---
lab:
  title: 练习 3 - 添加新命令
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft 365 Copilot'
---

# 练习 3 - 添加新命令

在本练习中，你将通过添加新命令来增强 Teams 消息扩展和 Copilot 插件。 虽然当前消息扩展有效地提供有关 Northwind 库存数据库中产品的信息，但它不提供与 Northwind 客户相关的信息。 你将引入一个与 API 调用关联的新命令，用于检索按用户指定的客户名称订购的产品。 本练习假设你已完成至少练习 1、2 和 3。 如果没有 Microsoft 365 Copilot 许可证，则可以跳过练习 4。

我们将通过以下任务来完成此操作：

1. **通过修改 Teams 应用清单来扩展消息扩展/插件用户界面**。 这包括引入新命令：**“companySearch”**。 注意到消息扩展的 UI 是自适应卡片，而对于 Copilot，它是 Copilot 聊天中的文本输入和输出。

1. **为 'companySearch' 命令创建处理程序**。 这将分析从消息路由代码传入的查询字符串，验证输入，并通过公司 API 调用产品搜索。 此任务还将使用返回的产品列表填充自适应卡片，并将其显示在消息或 Copilot 聊天 UI 中。

1. 更新命令**路由**代码，将新命令路由到上一任务中创建的处理程序。 当用户查询 Northwind 数据库 (**handleTeamsMessagingExtensionQuery**) 时，通过扩展 Bot Framework 调用的方法来执行此操作。 

1. **按公司实施产品搜索**，以返回该公司订购的产品列表。

1. **运行应用**并搜索由指定公司购买的产品。

## 任务 1 - 扩展消息扩展/插件用户界面 

1. 在 Visual Studio Code 中，从**工作文件夹**中打开 **manifest.json** 并紧接在 `discountSearch` 命令之后添加以下 json（**第 98 行**）。 使用此附加信息，可以添加到`commands`定义插件支持的命令列表的数组中。

   ```json
   {
       "id": "companySearch",
       "context": [
           "compose",
           "commandBox"
       ],
       "description": "Given a company name, search for products ordered by that company",
       "title": "Customer",
       "type": "query",
       "parameters": [
           {
               "name": "companyName",
               "title": "Company name",
               "description": "The company name to find products ordered by that company",
               "inputType": "text"
           }
       ]
   }
   ```

> [!NOTE] 
> ID **** 是 UI 与代码之间的连接。 此值在 discount\product\SearchCommand.ts**** 文件中定义为 COMMAND_ID****。 了解其中每个文件如何具有与 **id** 值对应的唯一 **COMMAND_ID**。

## 任务 2 - 为 'companySearch' 命令创建处理程序。

在本练习中，我们将复制一些现有代码，为命令创建新处理程序。 

1. 在 Visual Studio Code 的“工作目录”**** 下，导航到 .\src\messageExtensions**** 并将 '**productSearchCommand.ts**' 复制并粘贴到同一文件夹中以创建副本。 重命名此文件 **customerSearchCommand.ts**。

1. 将第 7 行更改为：

    ```typescript
    import { searchProductsByCustomer } from "../northwindDB/products";
    ```

1. 将第 10 行更改为 

   ```javascript
   const COMMAND_ID = "companySearch";
   ```



1. 将 **handleTeamsMessagingExtensionQuery** 的内容替换为：

   ```javascript
    {
       let companyName;
   
       // Validate the incoming query, making sure it's the 'companySearch' command
       // The value of the 'companyName' parameter is the company name to search for
       if (query.parameters.length === 1 && query.parameters[0]?.name === "companyName") {
           [companyName] = (query.parameters[0]?.value.split(','));
       } else { 
           companyName = cleanupParam(query.parameters.find((element) => element.name === "companyName")?.value);
       }
       console.log(`🍽️ Query #${++queryCount}:\ncompanyName=${companyName}`);    
   
       const products = await searchProductsByCustomer(companyName);
   
       console.log(`Found ${products.length} products in the Northwind database`)
       const attachments = [];
       products.forEach((product) => {
           const preview = CardFactory.heroCard(product.ProductName,
               `Customer: ${companyName}`, [product.ImageUrl]);
   
           const resultCard = cardHandler.getEditCard(product);
           const attachment = { ...resultCard, preview };
           attachments.push(attachment);
       });
       return {
           composeExtension: {
               type: "result",
               attachmentLayout: "list",
               attachments: attachments,
           },
       };
   }
   ```

> [!NOTE]
> 将在任务 4 中实现`searchProductsByCustomer`。

## 任务 3 - 更新命令路由

在此任务中，将`companySearch`命令路由到在上一个任务中实现的处理程序。

1. 打开 **searchApp.ts** 并在**第 10 行**中进行查找：

   ```javascript
   import discountedSearchCommand from "./messageExtensions/discountSearchCommand";
   ```

1. 将此添加为**第 11 行**：

   ```javascript
   import customerSearchCommand from "./messageExtensions/customerSearchCommand";
   ```

1. 在此语句下：

   ```javascript
         case discountedSearchCommand.COMMAND_ID: {
           return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
         }
   ```

1. 添加此语句：

   ```javascript
         case customerSearchCommand.COMMAND_ID: {
           return customerSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
         }
   ```

> [!NOTE]
> 明确调用插件基于 UI 的操作。 但是，Microsoft 365 Copilot 调用时，命令由 Copilot 业务流程协调程序触发。

## 任务 4 - 按公司实施产品搜索

在此任务中，你将**按公司名称**实施产品搜索，并返回公司的已订购产品列表。 查询输出的表如下所示：

| 表         | Find        | 通过以下方式查找    |
| ------------- | ----------- | ------------- |
| 客户      | 客户 ID | 客户名称 |
| 订单        | 订单 ID    | 客户 ID   |
| OrderDetail   | 产品     | 订单 ID      |

下面是流的工作原理： 

1. 使用**客户**表查找**客户 ID **和**客户名称**。 

1. 使用**客户 ID** 查询**订单**表，以检索关联的**订单 ID**。 

1. 对于每个**订单 ID**，查找 OrderDetail** 表中的关联产品**。 

1. 最后，返回按指定公司名称排序的产品列表。

现在，让我们修改 **products.ts** 文件以添加新的搜索查询。

1. 打开 **.\src\northwindDB\products.ts**

1. 更新第 1 行上的 `import` 语句，以包括 OrderDetail、Order 和 Customer。 它应如下所示：

   ```javascript
   import {
       TABLE_NAME, Product, ProductEx, Supplier, Category, OrderDetail,
       Order, Customer
   } from './model';
   ```

1. 在**第 8 行**下方：

   ```javascript
   import { getInventoryStatus } from '../adaptiveCards/utils';
   ```

       Add the new function `searchProductsByCustomer()`:

   ```javascript
   export async function searchProductsByCustomer(companyName: string): Promise<ProductEx[]> {

       let result = await getAllProductsEx();
   
       let customers = await loadReferenceData<Customer>(TABLE_NAME.CUSTOMER);
       let customerId="";
       for (const c in customers) {
           if (customers[c].CompanyName.toLowerCase().includes(companyName.toLowerCase())) {
               customerId = customers[c].CustomerID;
               break;
           }
       }
       
       if (customerId === "") 
           return [];

       let orders = await loadReferenceData<Order>(TABLE_NAME.ORDER);
       let orderdetails = await loadReferenceData<OrderDetail>(TABLE_NAME.ORDER_DETAIL);
       // build an array orders by customer id
       let customerOrders = [];
       for (const o in orders) {
           if (customerId === orders[o].CustomerID) {
               customerOrders.push(orders[o]);
           }
       }
       
       let customerOrdersDetails = [];
       // build an array order details customerOrders array
       for (const od in orderdetails) {
           for (const co in customerOrders) {
               if (customerOrders[co].OrderID === orderdetails[od].OrderID) {
                   customerOrdersDetails.push(orderdetails[od]);
               }
           }
       }

       // Filter products by the ProductID in the customerOrdersDetails array
       result = result.filter(product => 
           customerOrdersDetails.some(order => order.ProductID === product.ProductID)
       );

       return result;
   }
   ```

## 任务 5：运行应用！ 按公司名称搜索产品

现在，已准备好将示例作为 Microsoft 365 Copilot 插件进行测试。

1. 删除 Teams 中的 **Northwest Inventory **应用。 此任务是必需的，因为要更新清单。 清单更新要求重新安装应用。 执行此操作的最直接方法是首先将其从 Teams 中删除。

    1. 在 Teams 边栏中，选择三个点（...） 1️⃣。 应在应用程序列表中看到 **Northwind Inventory** 2️⃣。

    1. 右键单击 **Northwest Inventory** 图标，然后选择“卸载” 3️⃣。

        ![如何卸载 Northwind Inventory 的屏幕截图。](../media/3-01-uninstall-app.png)

1. 按 **F5**，在 Visual Studio Code 中使用 **Teams 中的调试 (Edge)** 配置文件启动应用。

1. 在 Teams 中，选择“聊天”****，然后选择 “Copilot” ****。 Copilot 应该是最顶级的选项。

1. 选择 **插件图标**，然后选择 **Northwind Inventory** 以启用插件。

1. 输入提示： 

   ```console
   What are the products ordered by 'Consolidated Holdings' in Northwind Inventory?
   ```

   终端输出显示 Copilot 理解查询并执行`companySearch`命令，传递 Copilot 提取的公司名称。

   ![Copilot 了解查询并执行 “companySearch” 命令的屏幕截图。](../media/3-08-terminal-query-output.png)

   下面是 Copilot 中的输出：

   ![Copilot 从命令生成结果的屏幕截图。](../media/3-07-response-customer-search.png)

下面是尝试的其他提示：

```console
What are the products ordered by 'Consolidated Holdings' in Northwind Inventory? Please list the product name, price and supplier in a table.
```

当然，还可以使用示例作为消息扩展来测试此新命令，就像我们在上一练习中所做的那样。 

1. 在 Teams 边栏中，转到 **聊天**部分，选择任何聊天或与同事开始新聊天。

1. 选择要 **+** 访问应用菜单的登录名。

1. 选择 **Northwind Inventory** 应用。

1. 请注意现在如何查看名为**客户**的新选项卡。

1. 搜索 **Consolidated Holdings**，并查看此公司订购的产品。 它们将与 Copilot 在上一个任务中返回的信息匹配。

    ![用作消息扩展的新命令的屏幕截图。](../media/3-08-customer-message-extension.png)

## 检查你的工作

完成练习后，你应该有一个新命令，用于在 Northwind Inventory 应用中按公司搜索订单。 还应能够成功地将插件与 Copilot 一起使用，并作为其他应用中的消息扩展。 

在下一个练习中，你将探索插件源代码和自适应卡片，详细了解如何构建应用以及如何进一步自定义应用。

[继续进行下一个练习...](./6-exercise-4-explore-plugin-source-code.md)