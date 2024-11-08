---
lab:
  title: 练习 4 - 探索插件源代码
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# 练习 4 - 探索插件源代码

在本练习中，你将查看应用程序代码，以便了解**消息扩展**的工作原理。

## 任务 1 - 检查清单

任何 Microsoft 365 应用程序的核心都是其应用程序清单。 可在其中提供 Microsoft 365 访问应用程序所需的信息。

在**工作目录中**，打开 **appPackackage/manifest.json** 文件。 此 JSON 文件与两个图标文件一起放在 zip 存档中，以创建应用程序包。 **图标**属性包括这些图标的路径。

```json
"icons": {
    "color": "Northwind-Logo3-192-${{TEAMSFX_ENV}}.png",
    "outline": "Northwind-Logo3-32.png"
},
```

请注意其中一个图标名称中的 `${{TEAMSFX_ENV}}` 标记。 Teams 工具包会将此标记替换为环境名称，例如**本地**或**开发**（用于开发中的 Azure 部署）。 因此，图标颜色将因环境而异。

### 应用程序说明

现在，请查看**名称**和**说明**。 请注意，**说明**相当长！ 这很重要，这样用户和 Copilot 都可以了解应用程序的功能以及何时使用。

```json
    "name": {
        "short": "Northwind Inventory",
        "full": "Northwind Inventory App"
    },
    "description": {
        "short": "App allows you to find and update product inventory information",
        "full": "Northwind Inventory is the ultimate tool for managing your product inventory. With its intuitive interface and powerful features, you'll be able to easily find your products by name, category, inventory status, and supplier city. You can also update inventory information with the app. \n\n **Why Choose Northwind Inventory:** \n\n Northwind Inventory is the perfect solution for businesses of all sizes that need to keep track of their inventory. Whether you're a small business owner or a large corporation, Northwind Inventory can help you stay on top of your inventory management needs. \n\n **Features and Benefits:** \n\n - Easy Product Search through Microsoft Copilot. Simply start by saying, 'Find northwind dairy products that are low on stock' \r - Real-Time Inventory Updates: Keep track of inventory levels in real-time and update them as needed \r  - User-Friendly Interface: Northwind Inventory's intuitive interface makes it easy to navigate and use \n\n **Availability:** \n\n To use Northwind Inventory, you'll need an active Microsoft 365 account . Ensure that your administrator enables the app for your Microsoft 365 account."
    },
```

### 机器人定义

向下滚动到 **composeExtensions**。 撰写扩展是消息扩展的历史术语；这是定义应用的消息扩展的位置。 消息扩展使用 Azure Bot Framework 进行通信；这提供了一个 Microsoft 365 和应用程序之间的快速安全的信道。 首次运行项目时，Teams 工具包注册了机器人，并将**机器人 ID** 放在此处。

```json
    "composeExtensions": [
        {
            "botId": "${{BOT_ID}}",
            "commands": [
                {
                    ...
```

### 命令定义

此消息扩展有两个命令，定义在 `commands` 数组中。 如果已完成上一个练习，则还会有第三个命令用于按公司名称进行搜索。 让我们暂时跳过第一个命令，因为它是最复杂的命令。 以下命令允许 Copilot （或用户）在 Northwind 类别中搜索折扣产品。 此命令接受单个参数，即 **categoryName**。

```json
{
    "id": "discountSearch",
    "context": [
        "compose",
        "commandBox"
    ],
    "description": "Search for discounted products by category",
    "title": "Discounts",
    "type": "query",
    "parameters": [
        {
            "name": "categoryName",
            "title": "Category name",
            "description": "Enter the category to find discounted products",
            "inputType": "text"
        }
    ]
},
```

现在，让我们回到第一个命令 **inventorySearch**，其中包含 5 个参数，允许更复杂的查询。

```json
{
    "id": "inventorySearch",
    "context": [
        "compose",
        "commandBox"
    ],
    "description": "Search products by name, category, inventory status, supplier location, stock level",
    "title": "Product inventory",
    "type": "query",
    "parameters": [
        {
            "name": "productName",
            "title": "Product name",
            "description": "Enter a product name here",
            "inputType": "text"
        },
        {
            "name": "categoryName",
            "title": "Category name",
            "description": "Enter the category of the product",
            "inputType": "text"
        },
        {
            "name": "inventoryStatus",
            "title": "Inventory status",
            "description": "Enter what status of the product inventory. Possible values are 'in stock', 'low stock', 'on order', or 'out of stock'",
            "inputType": "text"
        },
        {
            "name": "supplierCity",
            "title": "Supplier city",
            "description": "Enter the supplier city of product",
            "inputType": "text"
        },
        {
            "name": "stockQuery",
            "title": "Stock level",
            "description": "Enter a range of integers such as 0-42 or 100- (for >100 items). Only use if you need an exact numeric range.",
            "inputType": "text"
        }
    ]
},
```

Copilot 能够根据说明填充这些内容，消息扩展将返回由所有非空参数筛选的产品列表。

## 任务 2 - 检查机器人代码

现在打开 **src/searchApp.ts** 文件，其中包含使用 [Bot Builder SDK](https://learn.microsoft.com/azure/bot-service/index-bf-sdk) 与 Azure Bot Framework 通信的机器人代码。 请注意，机器人将扩展 SDK 类 **TeamsActivityHandler**。

```typescript
export class SearchApp extends TeamsActivityHandler {
  constructor() {
    super();
  }

  ...
```

### 消息扩展查询

应用程序可以通过重写 **TeamsActivityHandler** 的方法来处理来自 Microsoft 365 的消息（称为**活动**）。

其中第一个是**消息扩展查询**活动。 当用户在消息扩展中键入内容或 Copilot 调用该函数时，将调用此函数。 处理程序基于 **commandID** 调度查询。 这些与应用清单中使用的命令 ID 相同。

```typescript
  // Handle search message extension
  public async handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
  ): Promise<MessagingExtensionResponse> {

    switch (query.commandId) {
      case productSearchCommand.COMMAND_ID: {
        return productSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
      case discountedSearchCommand.COMMAND_ID: {
        return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
    }
  }
  ...
```

### 自适应卡片操作

应用需要处理的另一个活动类型是自适应卡片操作，例如当用户在自适应卡片上选择**更新库存**或**重新排序**时。 由于自适应卡片操作没有特定的方法，因此代码将重写 `onInvokeActivity()`，这是包含消息扩展查询的更广泛的活动类。 因此，代码会手动检查活动名称，并调度到相应的处理程序。 如果活动名称不是用于自适应卡片操作，则 `else` 子句将运行 `onInvokeActivity()` 的基本实现，此外，如果**调用**活动是查询，它将调用我们的 `handleTeamsMessagingExtensionQuery()` 方法。

```typescript
import {
  TeamsActivityHandler,
  TurnContext,
  MessagingExtensionQuery,
  MessagingExtensionResponse,
  InvokeResponse
} from "botbuilder";
import productSearchCommand from "./messageExtensions/productSearchCommand";
import discountedSearchCommand from "./messageExtensions/discountSearchCommand";
import revenueSearchCommand from "./messageExtensions/revenueSearchCommand";
import actionHandler from "./adaptiveCards/cardHandler";

export class SearchApp extends TeamsActivityHandler {
  constructor() {
    super();
  }

  // Handle search message extension
  public async handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
  ): Promise<MessagingExtensionResponse> {

    switch (query.commandId) {
      case productSearchCommand.COMMAND_ID: {
        return productSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
      case discountedSearchCommand.COMMAND_ID: {
        return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
    }

  }

  // Handle adaptive card actions
  public async onInvokeActivity(context: TurnContext): Promise<InvokeResponse> {
    let runEvents = true;
    // console.log (`🎬 Invoke activity received: ${context.activity.name}`);
    try {
      if(context.activity.name==='adaptiveCard/action'){
        switch (context.activity.value.action.verb) {
          case 'ok': {
            return actionHandler.handleTeamsCardActionUpdateStock(context);
          }
          case 'restock': {
            return actionHandler.handleTeamsCardActionRestock(context);
          }
          case 'cancel': {
            return actionHandler.handleTeamsCardActionCancelRestock(context);
          }
          default:
            runEvents = false;
            return super.onInvokeActivity(context);
        }
      } else {
          runEvents = false;
          return super.onInvokeActivity(context);
      }
    } 
```

## 任务 3 - 检查消息扩展命令代码

为了使代码更具模块化、可读性和可重用性，每个消息扩展命令都放置在自己的 TypeScript 模块中。 有关示例，请查看 **src/messageExtensions/discountSearchCommand.ts**。

首先，请注意该模块将导出一个常量 `COMMAND_ID`，其中包含与在应用清单中找到的相同 **commandID**，并允许 **searchApp.ts**中的 switch 语句正常工作。

然后，它提供一个函数 `handleTeamsMessagingExtensionQuery()`，用于处理**按类别查找折扣产品**的传入查询。

```typescript
async function handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
): Promise<MessagingExtensionResponse> {

    // Seek the parameter by name, don't assume it's in element 0 of the array
    let categoryName = cleanupParam(query.parameters.find((element) => element.name === "categoryName")?.value);
    console.log(`💰 Discount query #${++queryCount}: Discounted products with categoryName=${categoryName}`);

    const products = await getDiscountedProductsByCategory(categoryName);

    console.log(`Found ${products.length} products in the Northwind database`)
    const attachments = [];
    products.forEach((product) => {
        const preview = CardFactory.heroCard(product.ProductName,
            `Avg discount ${product.AverageDiscount}%<br />Supplied by ${product.SupplierName} of ${product.SupplierCity}`,
            [product.ImageUrl]);

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

请注意，`query.parameters` 数组中的索引可能与参数在清单中的位置不对应。 虽然这通常只是多参数命令的问题，但代码仍将基于参数名称获取值，而不是对索引进行硬编码。

清理参数（剪裁参数并处理有时 Copilot 假定“**\***”是匹配所有内容的通配符）后，代码将 Northwind 数据访问层调用为 `getDiscountedProductsByCategory()`。

然后循环访问产品，并为每个产品创建两张卡片：

- 作为**主**卡实现的_预览_卡。 这显示在用户界面的搜索结果和 Copilot 中的一些引文中。

- 作为包含所有详细信息的**自适应**卡实现的_结果_卡。

在下一个任务中，我们将查看自适应卡片代码并签出自适应卡片设计器。

## 任务 4 - 检查自适应卡片和相关代码

项目的自适应卡片位于 **src/adaptiveCards/** 文件夹中。 有 3 张卡片，每张卡片以 JSON 文件的形式实现。

- **editCard.json** - 这是消息扩展或 Copilot 引用显示的初始卡片。

- **successCard.json** - 当用户执行操作时，将显示此卡片以指示成功。 它大致与编辑卡相同，只不过它包含给用户的消息。

- **errorCard.json** - 如果操作失败，将显示此卡片。

让我们看看**自适应卡片设计器**中的编辑卡。 打开 Web 浏览器以访问 [https://adaptivecards.io](https://adaptivecards.io)，然后选择顶部的**设计器**选项。

![自适应卡片设计器的屏幕截图。](../media/5-01-adaptive-card-designer-01.png)

请注意数据绑定表达式，例如 `"text": "📦 ${productName}",`。 这会将数据中的 `productName` 属性与卡片上的文本绑定。

现在，选择 **Microsoft Teams** 作为主机应用程序 1️⃣。 将 **editCard.json** 的全部内容粘贴到卡片有效负载编辑器 2️⃣，并将 **sampleData.json** 的内容粘贴到示例数据编辑器 3️⃣。 示例数据与代码中提供的产品相同。 应将卡片视为呈现，但由于设计器无法显示其中一种自适应卡片格式而导致的一个小错误除外。

![根据 json 呈现的自适应卡片的 Copilot 屏幕截图。](../media/5-01-adaptive-card-designer-02.png)

在页面顶部附近，尝试更改**主题**和**模拟设备**，以查看卡片在深色主题或移动设备上的外观。 这是用于为示例应用程序生成自适应卡片的工具。

现在，返回到 Visual Studio Code，打开 **cardHandler.ts**。 从每个消息扩展命令调用函数 `getEditCard()` 以获取**结果**卡。 代码读取自适应卡片 JSON（被视为模板），然后将其绑定到产品数据。 结果是更多的 JSON - 与模板相同的卡片，其中填充了数据绑定表达式。 最后， `CardFactory` 模块用于将最终 JSON 转换为自适应卡片对象进行呈现。

```typescript
function getEditCard(product: ProductEx): any {

    var template = new ACData.Template(editCard);
    var card = template.expand({
        $root: {
            productName: product.ProductName,
            unitsInStock: product.UnitsInStock,
            productId: product.ProductID,
            categoryId: product.CategoryID,
            imageUrl: product.ImageUrl,
            supplierName: product.SupplierName,
            supplierCity: product.SupplierCity,
            categoryName: product.CategoryName,
            inventoryStatus: product.InventoryStatus,
            unitPrice: product.UnitPrice,
            quantityPerUnit: product.QuantityPerUnit,
            unitsOnOrder: product.UnitsOnOrder,
            reorderLevel: product.ReorderLevel,
            unitSales: product.UnitSales,
            inventoryValue: product.InventoryValue,
            revenue: product.Revenue,
            averageDiscount: product.AverageDiscount
        }
    });
    return CardFactory.adaptiveCard(card);
}
```

向下滚动后，将看到卡片上每个操作按钮的处理程序。 单击操作按钮时，卡片会提交数据，具体来说是 `data.txtStock`，即卡上的**数量**输入框；以及 `data.productId`，即在每个卡片操作中发送的数据，以便让代码知道要更新的产品。

```typescript
async function handleTeamsCardActionUpdateStock(context: TurnContext) {

    const request = context.activity.value;
    const data = request.action.data;
    console.log(`🎬 Handling update stock action, quantity=${data.txtStock}`);

    if (data.txtStock && data.productId) {

        const product = await getProductEx(data.productId);
        product.UnitsInStock = Number(data.txtStock);
        await updateProduct(product);

        var template = new ACData.Template(successCard);
        var card = template.expand({
            $root: {
                productName: product.ProductName,
                unitsInStock: product.UnitsInStock,
                productId: product.ProductID,
                categoryId: product.CategoryID,
                imageUrl: product.ImageUrl,
                ...
```

可以看到，代码获取这两个值，更新数据库，然后发送包含消息和更新数据的新卡片。

## 祝贺

已完成练习 5 和适用于 Microsoft 365 Copilot 消息扩展插件实验。 非常感谢做这些实验！

[继续学习实验室摘要...](./7-summary.md)
