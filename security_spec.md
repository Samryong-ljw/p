# Firestore Security Specification (`prompts`)

## 1. Data Invariants
- Every document in `prompts/{promptId}` must use a valid alphanumeric/hyphen/underscore document ID (`^[a-zA-Z0-9_-]+$`, 1-128 chars).
- Every prompt document must contain exactly the four keys: `title`, `category`, `content`, `createdAt` (no shadow or arbitrary extra keys).
- `title` must be a non-empty string up to 120 characters.
- `category` must be one of the 3 allowed categories: `'사진 합성'`, `'스타일 변환'`, `'캐릭터·포스터'`.
- `content` must be a non-empty string up to 20,000 characters (supporting 1,000+ character prompts while blocking 1MB payload abuse).
- `createdAt` must be a Firestore `timestamp` matching `request.time` on creation, and is immutable on update.
- Read access (`get`, bounded `list` with `limit <= 200`) is public so guests can browse, search, filter, view, and copy prompts without logging in.
- Write access (`create`, `update`, `delete`) is strictly restricted to the verified admin account (`request.auth != null && request.auth.token.email_verified == true && request.auth.token.email == 'leelove5323@gmail.com'`).

## 2. Adversarial Payloads Tested
1. Unauthenticated guest `create` / `update` / `delete` -> DENIED
2. Authenticated non-admin email `create` / `update` / `delete` -> DENIED
3. Spoofed admin email with `email_verified: false` -> DENIED
4. Missing required key (`title` omitted) -> DENIED
5. Shadow/ghost field injection (`isAdmin: true` or `extra: "x"`) -> DENIED
6. Invalid category enum (`"기타"`) -> DENIED
7. Oversized `content` (> 20,000 chars) -> DENIED
8. Empty `content` (`""`) -> DENIED
9. Client-spoofed `createdAt` on create (`createdAt != request.time`) -> DENIED
10. Mutating `createdAt` on update -> DENIED
11. Unbounded `list` query without `limit <= 200` -> DENIED
12. Unrecognized collection write (`/users/u1`) -> DENIED (default-deny catch-all)
