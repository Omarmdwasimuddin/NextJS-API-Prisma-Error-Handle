## NextJS API: Prisma error handle

#### lib/errors/handlePrismaError.ts
```bash
import { Prisma } from "@/app/generated/prisma";


export function handlePrismaError(error: unknown, requestId: string) {
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    switch (error.code) {
      case "P2002": return { status: 409, message: "Duplicate entry" };
      case "P2003": return { status: 400, message: "Invalid reference" };
      case "P2025": return { status: 404, message: "Not found" };
      default: return { status: 500, message: "Database error" };
    }
  }
  return { status: 500, message: "Internal server error" };
}
```
---
