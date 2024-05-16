# 🖇 Relação ManyToMany com typeorm

Neste repositório, explico detalhadamente como criar e configurar as tabelas necessárias em um banco de dados para suportar a relação Many-to-Many com typeorm e typescript. Neste exemplo, a relação foi entre pedidos (orders), clientes (customers) e produtos (products) em uma aplicação. Vamos examinar cada migração para entender como elas trabalham juntas para estabelecer essa relação complexa:

1. **CreateProducts**
```
import { MigrationInterface, QueryRunner, Table } from "typeorm"

export class CreateProducts1709088783975 implements MigrationInterface {
    public async up(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.createTable(new Table({
        name: "products",
        columns: [
          {
            name: "id",
            type: "uuid",
            isPrimary: true,
            generationStrategy: "uuid",
            default: "uuid_generate_v4()",
          },
          {
            name: "name",
            type: "varchar",
          },
          {
            name: "price",
            type: "decimal",
            precision: 10,
            scale: 2,
          },
          {
            name: "quantity",
            type: "int",
          },
          {
            name: "created_at",
            type: "timestamp",
            default: "now()"
          },
          {
            name: "updated_at",
            type: "timestamp",
            default: "now()"
          }
        ]
      }))
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.dropTable("products")
    }

}
```
- Esta migração cria a tabela `products`, que armazanará os produtos.
- Define colunas como `id` (chave primária), `name` (nome do produto), `price` (preço do produto), `quantity` (quantidade do produto) e timestamps (`created_at` e `updated_at`).

2. **CreateCustomers**
```
import { MigrationInterface, QueryRunner, Table } from "typeorm"

export class CreateCustomers1711321600884 implements MigrationInterface {

    public async up(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.createTable(
        new Table({
          name: "customers",
          columns: [
            {
              name: "id",
              type: "uuid",
              isPrimary: true,
              generationStrategy: "uuid",
              default: "uuid_generate_v4()"
            },
            {
              name: "name",
              type: "varchar"
            },
            {
              name: "email",
              type: "varchar",
            },
            {
              name: "created_at",
              type: "timestamp",
              default: "now()",
            },
            {
              name: "updated_at",
              type: "timestamp",
              default: "now()"
            },
          ]
        })
      )
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.dropTable("customers");
    }

}
```
- Esta migração cria a tabela `customers`, que armazanará os clientes.
- Define colunas como `id` (chave primária), `name` (nome do cliente), `email` (email do cliente) e timestamps (`created_at` e `updated_at`).

3. **CreateOrders**
   ```
   import { MigrationInterface, QueryRunner, Table } from "typeorm"

   export class CreateOrders17151421305 implements MigrationInterface {

    public async up(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.createTable(
        new Table({
          name: "orders",
          columns: [
            {
              name: "id",
              type: "uuid",
              isPrimary: true,
              generationStrategy: "uuid",
              default: "uuid_generate_v4()"
            },
            {
              name: "created_at",
              type: "timestamp",
              default: "now()",
            },
            {
              name: "updated_at",
              type: "timestamp",
              default: "now()"
            },
          ]
        })
      )
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.dropTable("orders");
    }
   }
   ``` 
   - Esta migração cria a tabela `orders` que armazenará os pedidos.
   - Define colunas como `id` (chave primária), `created_at` e `updated_at`.
   - O `id` é um UUID gerado automaticamente.
   - `created_at` e `updated_at` são campos de timestamp definidos com valores padrão.

4. **CreateOrdersProducts**
```
import { MigrationInterface, QueryRunner, Table } from "typeorm"

export class CreateOrdersProducts1712451470610 implements MigrationInterface {

    public async up(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.createTable(
        new Table({
          name: "orders_products",
          columns: [
            {
              name: "id",
              type: "uuid",
              isPrimary: true,
              generationStrategy: "uuid",
              default: "uuid_generate_v4()"
            },
            {
              name: "price",
              type:"decimal",
              precision: 10,
              scale: 2,
            },
            {
              name: "quantity",
              type: "int",
            },
            {
              name: "created_at",
              type: "timestamp",
              default: "now()",
            },
            {
              name: "updated_at",
              type: "timestamp",
              default: "now()"
            },
          ]
        })
      )
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.dropTable("orders_products");
    }
}

```
   - Esta migração cria a tabela `orders_products` que servirá como uma tabela intermediária entre pedidos e produtos para implementar a relação Many-to-Many.
   - Define colunas como `id` (chave primária), `price` (preço do produto no pedido), `quantity` (quantidade do produto no pedido) e timestamps (`created_at` e `updated_at`).

5. **AddCustomerIdToOrders**
```
import { MigrationInterface, QueryRunner, TableColumn, TableForeignKey } from "typeorm"

export class AddCustomerIdToOrders1712432736009 implements MigrationInterface {

    public async up(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.addColumn(
        "orders",
        new TableColumn({
          name: "customer_id",
          type: "uuid",
          isNullable: true,
        })
      )
      await queryRunner.createForeignKey(
        "orders",
        new TableForeignKey({
          name: "OrdersCustomer",
          columnNames: ["customer_id"],
          referencedTableName: "customers",
          referencedColumnNames: ["id"],
          onDelete: "SET NULL",
        })
      )
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.dropForeignKey("orders", "OrdersCustomer")
      await queryRunner.dropColumn("orders", "customer_id");
    }
}
```
   - Esta migração adiciona a coluna `customer_id` à tabela `orders`, que será usada para associar cada pedido a um cliente.
   - Define a coluna como um UUID e permite valores nulos (`isNullable: true`).
   - Cria uma chave estrangeira (`OrdersCustomer`) que referencia a tabela `customers` pelo `id`, indicando que um pedido pode estar associado a um cliente.

6. **AddOrderIdToOrdersProducts**
```
import { MigrationInterface, QueryRunner, TableColumn, TableForeignKey } from "typeorm"

export class AddOrderIdToOrdersProducts1712451849377 implements MigrationInterface {

    public async up(queryRunner: QueryRunner): Promise<void> {
       await queryRunner.addColumn(
        "orders_products",
        new TableColumn({
          name: "order_id",
          type: "uuid",
          isNullable: true,
        })
      )
      await queryRunner.createForeignKey(
        "orders_products",
        new TableForeignKey({
          name: "OrdersProductsOrder",
          columnNames: ["order_id"],
          referencedTableName: "orders",
          referencedColumnNames: ["id"],
          onDelete: "SET NULL",
        })
      )
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.dropForeignKey("orders_products", "OrdersProductsOrder");
      await queryRunner.dropColumn("orders_products", "order_id");
    }
}
```
   - Esta migração adiciona a coluna `order_id` à tabela `orders_products`, permitindo que cada registro nesta tabela esteja vinculado a um pedido específico.
   - Define a coluna como um UUID e permite valores nulos (`isNullable: true`).
   - Cria uma chave estrangeira (`OrdersProductsOrder`) que referencia a tabela `orders` pelo `id`, estabelecendo a relação entre um produto em um pedido e o pedido em si.

7. **AddProductIdToOrdersProducts**
```
import { MigrationInterface, QueryRunner, TableColumn, TableForeignKey } from "typeorm"

export class AddProductIdToOrdersProducts1712452118632 implements MigrationInterface {

    public async up(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.addColumn(
        "orders_products",
        new TableColumn({
          name: "product_id",
          type: "uuid",
          isNullable: true,
        })
      )
      await queryRunner.createForeignKey(
        "orders_products",
        new TableForeignKey({
          name: "OrdersProductsProduct",
          columnNames: ["product_id"],
          referencedTableName: "products",
          referencedColumnNames: ["id"],
          onDelete: "SET NULL",
        })
      )
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.dropForeignKey("orders_products", "OrdersProductsProduct");
      await queryRunner.dropColumn("orders_products", "product_id");
    }

}
```
   - Esta migração adiciona a coluna `product_id` à tabela `orders_products`, permitindo que cada registro nesta tabela esteja vinculado a um produto específico.
   - Define a coluna como um UUID e permite valores nulos (`isNullable: true`).
   - Cria uma chave estrangeira (`OrdersProductsProduct`) que referencia a tabela `products` pelo `id`, estabelecendo a relação entre um produto em um pedido e o produto em si.

8. **AddOrderFieldtoOrders**
```
import { MigrationInterface, QueryRunner, TableColumn } from 'typeorm';

export class AddOrderFieldtoOrders1619889809717 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.addColumn(
      'orders',
      new TableColumn({
        name: 'order',
        type: 'int',
        isGenerated: true,
        generationStrategy: 'increment',
      }),
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('orders', 'order');
  }
}
```
   - Esta migração adiciona o campo `order` à tabela `orders`, que é o número de identificação único para cada pedido.
   - Define o campo como um número inteiro (`int`) e configura para ser gerado automaticamente com base em uma estratégia de incremento (`generationStrategy: 'increment'`).

Em resumo, essas migrações juntas formam a estrutura necessária no banco de dados para permitir a criação, associação e consulta de pedidos, clientes e produtos em sua aplicação, estabelecendo relacionamentos complexos através do TypeORM e garantindo a integridade dos dados. As chaves estrangeiras e as tabelas intermediárias (`orders_products`) são fundamentais para representar corretamente a relação Many-to-Many entre pedidos e produtos.
