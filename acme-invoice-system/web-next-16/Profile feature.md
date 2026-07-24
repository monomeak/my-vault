---
tags: [nextjs, react-query, feature-architecture]
aliases: [User profile]
created: 2026-07-17
---

# Profile feature

> [!abstract] What it does
> Reads and edits the logged-in user's profile against DummyJSON's `/users/:id`, sharing role logic with [[Auth feature]].

Related: [[React Query]] · [[Auth feature]] · [[Feature-based architecture]] · [[Zod]]

---

## Folder structure

```
src/features/profile/
├── api/
│   └── profile-api.ts       # fetchProfileById, updateProfile
├── hooks/
│   └── use-profile.ts       # useProfile (read) + useUpdateProfile (write) — two separate hooks
├── lib/
│   └── query-keys.ts        # profileKeys.all / .detail(id)
├── mappers/
│   └── profile.mapper.ts    # DTO -> UserProfile, reuses role.mapper.ts from Auth feature
├── schemas/
│   └── profile.schema.ts    # Zod validation for the API response
├── types/
│   ├── dummy-user.ts        # raw DummyJSON shape
│   └── profile.ts           # UserProfile, EditableProfileField
└── components/
    ├── profile-header.tsx       # read-only: avatar, name, title
    ├── profile-information.tsx  # read-only: email, phone, address
    └── profile-form.tsx         # editable fields, uses useUpdateProfile directly
```

---

## Why read and write are two separate hooks

```typescript
// src/features/profile/hooks/use-profile.ts
export function useProfile(userId: number) {
  return useQuery<UserProfile>({
    queryKey: profileKeys.detail(userId),
    queryFn: async () => mapToUserProfile(await fetchProfileById(userId)),
    staleTime: 60 * 1000,
  });
}

export function useUpdateProfile(userId: number) {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: async (updates: Partial<UserProfile>) =>
      updateProfile(userId, mapToDto(updates)),
    onSuccess: (updatedDto) => {
      queryClient.setQueryData<UserProfile>(
        profileKeys.detail(userId),
        (old) => (old ? { ...old, ...updatedDto } : old)
      );
    },
  });
}
```

> [!tip] Components that only render don't need mutation state
> `profile-header.tsx` and `profile-information.tsx` only call `useProfile` (the query). Only `profile-form.tsx` needs `useUpdateProfile` (the mutation). Bundling both into one hook would force every consumer to pull in mutation state it never uses.

---

## `EditableProfileField` — deriving safety from the domain type

```typescript
// src/features/profile/types/profile.ts
export type EditableProfileField = Exclude<
  keyof UserProfile,
  "id" | "image" | "role" | "company" | "address"
>;
```

> [!question] Why derive instead of hand-listing
> If a new field is added to `UserProfile` later, TypeScript forces a decision: is it editable or excluded? Nothing can silently fall through the cracks the way it could with a manually maintained array of field names.

---

## Cache merge instead of refetch after save

DummyJSON's `PATCH /users/:id` only echoes back the fields that were sent, not the full updated user. So instead of refetching the whole profile after a save, the mutation's `onSuccess` merges the partial response into the existing cache entry directly — see [[React Query]] → "Cache updates after a mutation: seed vs invalidate."

---

## Cross-feature import: is this okay?

```typescript
// src/features/profile/mappers/profile.mapper.ts
import { mapApiRoleToAppRole, parseApiRole } from "@/features/auth/mappers/role.mapper";
```

> [!warning] Worth watching as the app grows
> This is a deliberate cross-feature import — roles are fundamental enough to share rather than duplicate. But if more of these cross-imports appear between features, that's the signal to extract shared concepts (roles, permissions) into a `src/shared/` or `src/entities/` layer both features import from, rather than importing directly from each other. See [[Feature-based architecture]].

---

## See also
- [[Auth feature]] — source of the shared `role.mapper.ts`
- [[React Query]] — `useQuery` + `useMutation` + cache-merge pattern used here
- [[Zod]] — `profile.schema.ts` validates the DummyJSON response shape
