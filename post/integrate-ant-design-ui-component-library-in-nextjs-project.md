# 在Nextjs项目中集成Ant Design UI组件库

Ant Design是一款非常优秀的UI组件库，在中后台的使用中相当广泛。

## 创建带有Ant Design UI组件的项目模版

如果是一个新的项目，可以通过下面的指令创建带有Ant Design UI组件的[项目模版](https://github.com/vercel/next.js/tree/canary/examples/with-ant-design)

```bash
npx create-next-app --example with-ant-design with-ant-design-app
```

## 现有项目集成

如果是现有的项目中希望集成Ant Design，则可以通过下面的方式进行

- 创建演示项目

```bash
npx create-next-app next-antd
```

- 进入项目根目录

```bash
cd next-antd
```

- 安装antd相关依赖

```bash
npm install antd --save
```

- 在**`pages/_app.js`**文件的最顶部手动导入**`antd.css`**全局CSSS文件

```bash
$ vim pages/_app.js
import 'antd/dist/antd.css'
```

- 创建`pages/antd.js`演示页面，放入一些`antd`组件

```bash
$ vim pages/antd.js
import { Button, Space, DatePicker } from 'antd';

export default function AntdDemo() {
    const onChange = () => { };
    return (
        <div style={{ padding: 100 }}>
            <Space direction="vertical">
                <Button type="primary">Primary Button</Button>
                <Button type="ghost">Ghost Button</Button>
                <DatePicker onChange={onChange} />
            </Space>
        </div>
    );
}
```

- 运行并查看

```bash
npm run dev
```

最后页面看起来应该是这样

![Untitled](/images/2022/04/18/1.png)