# Thought 模块设计

## 定位

Thought（想法）是唯一属于**流体域**的认知构件——从 stdin 输入的结构化产物，尚未定型入库。

Thought 是系统的入口：用户输入原始文本，clarify 产出一个或多个 Thought。

## 数据模型

```
Thought {
    id: Uuid,
    title: String,          // 概括
    description: String,    // 用户原始文本
    created_at: DateTime,
}
```

最小设计——Thought 不包含 Situation / Intention / Schema 的任何字段。它只负责"用户说了什么"，不负责"用户说的是什么"。后者的结构化由下游命令处理。

## CLI 中的角色

作为独立 crate，提供 Thought 类型的序列化/反序列化。

```
qtcloud-think-thought/        # dependencies: 无
```

Thought 是唯一一个不在 gallery 中持久化的类型。它的生命周期始于 stdin，终于 stdout（distill 产出 YAML 后即完成使命）。

## 开放问题

1. Thought 是否需要关联到已存在的 Situation？如果有模糊关联（"这条想法可能属于组织管理"），关联信息存在哪里？
