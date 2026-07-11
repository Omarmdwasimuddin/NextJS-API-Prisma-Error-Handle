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

#### app/api/order/create
```bash
import { NextRequest, NextResponse } from "next/server";
import prisma from "@/lib/prisma";
import { log } from "@/lib/logger"
import { createOrderSchema } from "@/lib/validations/order.schema";
import { handlePrismaError } from "@/lib/errors/handlePrismaError"

export async function POST(request: NextRequest) {
    const requestId = crypto.randomUUID();
    try {
        log.info(
            {
                requestId,
                path: request.nextUrl.pathname,
                method: request.method,
            },
            "Order data receieved"
        )

        const { searchParams } = new URL(request.url);
        const userID = searchParams.get('id')

        if (!userID) {
            log.warn(
                {
                    requestId,
                    userID,
                },
                "User ID is required!"
            )

            return NextResponse.json(
                {status: "fail", requestId, message: "User ID is required"},
                {status: 400}
            )
        }

        const jsonBody = await request.json();
        const validation = createOrderSchema.safeParse(jsonBody)

        if (!validation.success) {
            log.warn(
                {
                    errors: validation.error.flatten().fieldErrors,
                },
                "Invalid Input"
            )

            return NextResponse.json(
                {status: "fail", requestId, message: "Invalid Input", errors: validation.error.flatten().fieldErrors},
                {status: 400}
            )
        }

        const createOrder = await prisma.order.create({
            data: {
                ...validation.data,
                userId: userID,
            }
        })

        log.info(
            {
                requestId,
                orderID: createOrder.id,
                productID: createOrder.productId,
                customerName: createOrder.customerName,
            },
            "Create order successfully!"
        )

        return NextResponse.json(
            {status: "success", requestId, message: "Create order successfully!", data: createOrder},
            {status: 201}
        )
    } catch (error) {
        const { status, message } = handlePrismaError(error, requestId);
        log.error(
            { 
                requestId,
                err: error instanceof Error ? error.message : String(error),
             },
            "Failed order creating..."
        )
        return NextResponse.json(
            {status: "fail", requestId, message: "Failed order creating..."},
            {status: 500}
        )
    }
}
```
---
