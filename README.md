# NestJS Swagger DTO generator

Generates separate TypeScript files with DTO classes and enums from a `data-contracts.ts` file produced by [swagger-typescript-api](https://github.com/acacode/swagger-typescript-api).

Each DTO class is decorated with `@ApiProperty` (from `@nestjs/swagger`) so that the generated entities are fully documented in Swagger/OpenAPI. Field descriptions, examples and `@deprecated` markers are preserved from the source `data-contracts`, and cross-entity references are resolved as `type: () => SomeEntity` / `enum: SomeEnum` / `isArray: true`. Generated files are marked with `/* eslint-disable */` and `/* tslint:disable */` headers so they stay out of linting.

## Installation

```
npm i -D @kovalenko/nest-swagger-dto-generator
```

## Usage

Run it against a swagger-typescript-api `data-contracts.ts` file:

```bash
nest-swagger-dto-generator ./src/client/data-contracts.ts ./src/dto
```

### Options

| Option                  | Description                                                              |
| ----------------------- | ------------------------------------------------------------------------ |
| `--only Name1,Name2`    | Comma-separated list of entities to export; other entities are skipped   |
| `--clean`               | Wipe the output folder before generating                                 |
| `--help`                | Show usage                                                               |

### Naming

Files are named with `kebab-case`:
- `OrderDto` -> `order.dto.ts`
- `OrderStatus` -> `order-status.enum.ts`
- `PaymentMethod` -> `payment-method.enum.ts`

## Example

The package ships with an abstract OpenAPI schema at `examples/openapi.yaml` — an "Orders API" with enums and related entities:

```yaml
openapi: 3.0.0
info:
  title: Orders API
  description: Abstract service used to demonstrate the generator
  version: "1.0.0"
paths:
  /orders:
    get:
      operationId: getOrders
      summary: List orders
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/OrderDto"
components:
  schemas:
    OrderStatus:
      type: string
      x-enumNames:
        - Pending
        - Placed
        - Shipped
        - Delivered
        - Cancelled
      enum:
        - pending
        - placed
        - shipped
        - delivered
        - cancelled
    PaymentMethod:
      type: string
      x-enumNames:
        - Card
        - Cash
        - Invoice
      enum:
        - card
        - cash
        - invoice
    OrderItemDto:
      type: object
      required: [id, name, price]
      properties:
        id:
          type: integer
          description: Item id
        name:
          type: string
          description: Item name
        price:
          type: number
          format: float
          description: Unit price
        quantity:
          type: integer
          description: Quantity
          example: 3
    CustomerDto:
      type: object
      required: [id, fullName, email]
      properties:
        id:
          type: integer
          description: Customer id
        fullName:
          type: string
          description: Full name
        email:
          type: string
          format: email
          description: Email address
        loyaltyPoints:
          type: integer
          description: Loyalty points balance
          example: 1240
    OrderDto:
      type: object
      required: [id, total, status, customer]
      properties:
        id:
          type: integer
          description: Order id
        createdAt:
          type: string
          format: date-time
          description: Creation timestamp
        total:
          type: number
          format: float
          description: Total amount
        status:
          $ref: "#/components/schemas/OrderStatus"
        paymentMethod:
          $ref: "#/components/schemas/PaymentMethod"
        customer:
          $ref: "#/components/schemas/CustomerDto"
        items:
          type: array
          description: Ordered items
          items:
            $ref: "#/components/schemas/OrderItemDto"
```

### 1. Generate the contracts

```bash
npx swagger-typescript-api generate -p ./examples/openapi.yaml -o ./src/client --modular
```

It produces `./src/client/data-contracts.ts` with enums and interfaces:

```typescript
export enum OrderStatus {
  Pending = "pending",
  Placed = "placed",
  Shipped = "shipped",
  Delivered = "delivered",
  Cancelled = "cancelled",
}

export interface OrderDto {
  /** Order id */
  id: number;
  /** Creation timestamp @format date-time */
  createdAt?: string;
  /** Total amount @format float */
  total: number;
  status: OrderStatus;
  paymentMethod?: PaymentMethod;
  customer: CustomerDto;
  /** Ordered items */
  items?: OrderItemDto[];
}
```

Enum member names are taken from `x-enumNames` in the schema (otherwise they fall back to the capitalized value).

### 2. Generate the DTOs

```bash
nest-swagger-dto-generator ./src/client/data-contracts.ts ./src/dto
```

```
ENUM      PaymentMethod -> ./src/dto/payment-method.enum.ts
ENUM      OrderStatus   -> ./src/dto/order-status.enum.ts
DTO       CustomerDto   -> ./src/dto/customer.dto.ts
DTO       OrderDto      -> ./src/dto/order.dto.ts
DTO       OrderItemDto  -> ./src/dto/order-item.dto.ts

Exported 5 entities to ./src/dto
```

`./src/dto/order.dto.ts`:

```typescript
/* eslint-disable */
/* tslint:disable */
// This file was generated by nest-swagger-dto-generator. Do not edit.
import {ApiProperty} from '@nestjs/swagger';

import {OrderDto as OrderDtoInterface} from '../client/data-contracts';
import {CustomerDto} from './customer.dto';
import {OrderItemDto} from './order-item.dto';
import {OrderStatus} from './order-status.enum';
import {PaymentMethod} from './payment-method.enum';

export class OrderDto implements OrderDtoInterface {
  @ApiProperty({description: 'Order id'})
  id: number;

  @ApiProperty({description: 'Creation timestamp @format date-time', required: false})
  createdAt?: string;

  @ApiProperty({description: 'Total amount @format float'})
  total: number;

  @ApiProperty({enum: OrderStatus, enumName: 'OrderStatus'})
  status: OrderStatus;

  @ApiProperty({required: false, enum: PaymentMethod, enumName: 'PaymentMethod'})
  paymentMethod?: PaymentMethod;

  @ApiProperty({type: () => CustomerDto})
  customer: CustomerDto;

  @ApiProperty({description: 'Ordered items', required: false, type: () => OrderItemDto, isArray: true})
  items?: OrderItemDto[];
}
```

## License

[MIT](LICENSE)