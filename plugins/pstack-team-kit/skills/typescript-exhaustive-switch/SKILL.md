---
name: typescript-exhaustive-switch
description: Use exhaustive switch handling for TypeScript unions and enums. Use when writing, editing, or reviewing TypeScript switch statements over discriminated unions or enums.
paths: ["**/*.ts", "**/*.tsx"]
---

# TypeScript exhaustive switch

In switch statements over discriminated unions or enums, use a `never` check in the default case so newly added variants cause compile-time failures until handled.
