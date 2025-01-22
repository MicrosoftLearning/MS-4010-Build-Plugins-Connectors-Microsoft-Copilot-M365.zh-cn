---
lab:
  title: 练习 1 - 下载项目并检查文件
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# 练习 1 - 下载项目并检查文件

通过操作扩展声明性代理可支持该代理实时检索和更新存储在外部系统中的数据。 使用 API 插件，你可以通过其 API 连接到外部系统以检索和更新信息。

### 练习用时

- **估计完成时间：** 10 分钟

## 任务 1 - 下载初学者项目

首先下载示例项目。 在 Web 浏览器中：

1. 导航到 [https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript](https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript)。
    1. 按照步骤，[将存储库源代码下载](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view)到计算机。
    1. 将下载的 zip 文件的内容解压缩到 **Documents 文件夹**。
    1. 在 Visual Studio Code 中打开  文件夹，

示例项目是一个 Teams 工具包项目，其中包含声明性代理和 Azure Functions 上运行的匿名 API。 此声明性代理与使用 Teams 工具包新建的声明性代理相同。 API 属于虚构的意大利餐厅，允许浏览今天的菜单并下订单。

## 任务 2 - 检查 API 定义

首先，查看意大利餐厅 API 的 API 定义。

在 Visual Studio Code 中：

1. 在“**资源管理器**”视图中，打开 **appPackage/apiSpecificationFile/ristorante.yml** 文件。 该文件是 OpenAPI 规范，介绍意大利餐厅的 API。
1. 找到 **servers.url** 属性

    ```yaml
    servers:
      - url: http://localhost:7071/api
        description: Il Ristorante API server
    ```

    请注意，它指向本地运行 Azure Functions 时与标准 URL 匹配的本地 URL。

1. 找到 **paths** 属性，其中包含两个操作：**/dishes** 用于检索今日菜单，以及 **/orders** 用于下订单。

    > [!IMPORTANT]
    > 请注意，每个操作都包含可唯一标识 API 规范中操作的 **operationId** 属性。 Copilot 要求每个操作都有唯一 ID，以便它知道针对特定用户提示应调用哪个 API。

## 任务 3 - 检查 API 实现

接下来，查看在本练习中使用的示例 API。

在 Visual Studio Code 中：

1. 在“**资源管理器**”视图中，打开 **src/data.json** 文件。 该文件包含意大利餐厅的虚拟菜单项。 每种菜肴都包含：

    - 名称，
    - 介绍，
    - 图像链接，
    - 价格，
    - 其在哪个菜品中供应，
    - 类型（菜肴或饮料），
    - （可选）过敏原列表

    在本练习中，API 使用此文件作为其数据源。
1. 接下来，展开 **src/functions** 文件夹。 请注意两个名为 **dishes.ts** 和 **placeOrder.ts** 的文件。 这些文件包含对 API 规范中定义的两个操作的实现。
1. 打开 **src/functions/dishes.ts** 文件。 花点时间查看 API 的工作原理。 它首先从 **src/functions/data.json** 文件加载示例数据。

    ```typescript
    import data from "../data.json";
    ```

    接下来，它会在不同的查询字符串参数中查找调用 API 的客户端可能传递的筛选条件。

    ```typescript
    const course = req.query.get('course');
    const allergensString = req.query.get('allergens');
    const allergens: string[] = allergensString ? allergensString.split(",") : [];
    const type = req.query.get('type');
    const name = req.query.get('name');
    ```

    根据请求指定的筛选条件，API 筛选数据集并返回响应。

1. 接下来，检查 **src/functions/placeOrder.ts** 文件中定义用于下订单的 API。 API 从引用示例数据开始。 然后，它定义客户端在请求正文中发送的订单的形状。

    ```typescript
    interface OrderedDish {
      name?: string;
      quantity?: number;
    }
    interface Order {
      dishes: OrderedDish[];
    }
    ```

    当 API 处理请求时，会首先检查请求是否包含正文，以及正文是否具有正确的形状。 否则，它会拒绝请求，并出现 “400 错误请求”错误。

    ```typescript
    let order: Order | undefined;
    try {
      order = await req.json() as Order | undefined;
    }
    catch (error) {
      return {
        status: 400,
        jsonBody: { message: "Invalid JSON format" },
      } as HttpResponseInit;
    }
    if (!order.dishes || !Array.isArray(order.dishes)) {
      return {
        status: 400,
        jsonBody: { message: "Invalid order format" }
      } as HttpResponseInit;
    }
    ```

    接下来，API 将请求解析为菜单上的菜肴，并计算总价格。

    ```typescript
    let totalPrice = 0;
    const orderDetails = order.dishes.map(orderedDish => {
      const dish = data.find(d => d.name.toLowerCase().includes(orderedDish.name.toLowerCase()));
      if (dish) {
        totalPrice += dish.price * orderedDish.quantity;
        return {
          name: dish.name,
          quantity: orderedDish.quantity,
          price: dish.price,
        };
      }
      else {
        context.error(`Invalid dish: ${orderedDish.name}`);
        return null;
      }
    });
    ```

    > [!IMPORTANT]
    > 请注意 API 期望客户端按其名称的一部分而不是其 ID 来指定菜肴。 这是有目的的，因为大型语言模型使用文字比数字更好。 此外，在调用 API 从而下订单之前，Copilot 具有作为用户提示的一部分，随时可用的菜肴名称。 如果 Copilot 必须按其 ID 引用菜肴，首先需检索需要其他 API 请求和 Copilot 现在无法执行的操作。

    API 准备就绪后，将返回总价格、补全订单 ID 和状态的响应。

    ```typescript
    const orderId = Math.floor(Math.random() * 10000);
    return {
      status: 201,
      jsonBody: {
        order_id: orderId,
        status: "confirmed",
        total_price: totalPrice,
      }
    } as HttpResponseInit;
    ```

