---
title: 联机托管说明
permalink: index.html
layout: home
---

此存储库包含 Microsoft 课程 [DP-420 使用 Microsoft Azure Cosmos DB 设计和实现云原生应用程序][course-description]和 [Microsoft Learn 上的等效自定进度模块][learn-collection]的动手实验室练习。 这些练习与学习材料配合使用，让你能够使用其描述的技术进行练习。

> &#128221; 若要完成这些练习，你需要一个 Microsoft Azure 订阅。 可在 [https://azure.microsoft.com][azure] 注册免费试用版。

## 实验室

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %}
| 模块 | 实验室 |
| --- | --- |
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
