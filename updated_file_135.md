# 场景：影视（基于tugraph-db的产品内置场景上手体验）

> 此文档主要介绍 影视场景Demo 的使用方法。

## 1.建模和数据导入

完成登录后，点击`新建图项目`，选择movie数据，填写图项目配置，系统会自动完成Movie场景图项目创建。其余场景也可用相同方式创建。

Movie 场景图：

<img src="https://tugraph-web-static.oss-cn-beijing.aliyuncs.com/%E6%96%87%E6%A1%A3/1.Guide/2.quick-start.png" alt="movie_schema" style="zoom: 25%;" />

| 标签          | 类型  | 说明                                  |
|-------------|-----|-------------------------------------|
| movie       | 实体  | 表示某一部具体的影片，比如"阿甘正传"。                |
| person      | 实体  | 表示个人，对影片来说可能是演员、导演，或编剧。             |
| genre       | 实体  | 表示影片的类型，比如剧情片、恐怖片。                  |
| keyword     | 实体  | 表示与影片相关的一些关键字，比如"拯救世界"、"虚拟现实"、"地铁"。 |
| user        | 实体  | 表示观影的用户。                            |
| produce     | 关系  | 表示影片的出品人关系。                         |
| acted_in    | 关系  | 表示演员出演了哪些影片。                        |
| direct      | 关系  | 表示影片的导演是谁。                          |
| write       | 关系  | 表示影片的编剧关系。                          |
| has_genre   | 关系  | 表示影片的类型分类。                          |
| has_keyword | 关系  | 表示影片的一些关键字，即更细分类的标签。                |
| rate        | 关系  | 表示用户对影片的打分。                         |
表格内容描述:
该表格包含三列，分别为“标签”、“类型”和“说明”。

逐行描述数据内容如下：
1. 标签“movie”是一个实体，表示某一部具体的影片，比如“阿甘正传”。
2. 标签“person”也是一个实体，表示个人，可能是演员、导演或编剧。
3. 标签“genre”为实体，表示影片的类型，比如剧情片或恐怖片。
4. 标签“keyword”为实体，表示与影片相关的一些关键字，例如“拯救世界”、“虚拟现实”或“地铁”。
5. 标签"user"为实体，表示观影的用户。
6. 标签“produce”是一个关系，表示影片的出品人关系。
7. 标签“acted_in”也是一个关系，表示演员出演了哪些影片。
8. 标签“direct”表示影片的导演是谁。
9. 标签“write”是一个关系，表示影片的编剧关系。
10. 标签“has_genre”是关系，表示影片的类型分类。
11. 标签“has_keyword”表示影片的一些关键字，即更细分类的标签。
12. 标签“rate”是一个关系，表示用户对影片的打分。

整体内容的概要总结是：该表格定义了与影片相关的多种实体（如影片、个人、类型和关键字）及其之间的关系（如出品人、演员、导演、编剧、类型分类、关键字和用户评分）。通过这些标签，可以清晰地描述影片的基本信息和相关的用户交互。

## 2.查询示例

### 2.1.示例一

查询影片 'Forrest Gump' 的所有演员，返回影片和演员构成的子图。

```
MATCH (m:movie {title: 'Forrest Gump'})<-[:acted_in]-(a:person) RETURN a, m
```

### 2.2.示例二

查询影片 'Forrest Gump' 的所有演员，列出演员在影片中扮演的角色。

```
MATCH (m:movie {title: 'Forrest Gump'})<-[r:acted_in]-(a:person) RETURN a.name,r.role
```

### 2.3.示例三

查询 Michael 所有评分低于 3 分的影片。

```
MATCH (u:user {login: 'Michael'})-[r:rate]->(m:movie) WHERE r.stars < 3 RETURN m.title, r.stars
```

### 2.4.示例四

查询和 Michael 有相同讨厌的影片的用户，讨厌标准为评分小于三分。

```
MATCH (u:user {login: 'Michael'})-[r:rate]->(m:movie)<-[s:rate]-(v) WHERE r.stars < 3 AND s.stars < 3 RETURN u, m, v
```

### 2.5.示例五

给 Michael 推荐影片，方法为先找出和 Michael 讨厌同样影片的用户，再筛选出这部分用户喜欢的影片。

```
MATCH (u:user {login: 'Michael'})-[r:rate]->(m:movie)<-[s:rate]-(v)-[r2:rate]->(m2:movie) WHERE r.stars < 3 AND s.stars < 3 AND r2.stars > 3 RETURN u, m, v, m2
```

### 2.6.示例六

查询 Michael 的好友们喜欢的影片。

```
MATCH (u:user {login: 'Michael'})-[:is_friend]->(v:user)-[r:rate]->(m:movie) WHERE r.stars > 3 RETURN u, v, m
```

### 2.7.示例七

通过查询给'Forrest Gump'打高分的人也喜欢哪些影片，给喜欢'Forrest Gump'的用户推荐类似的影片。

```
MATCH (m:movie {title:'Forrest Gump'})<-[r:rate]-(u:user)-[r2:rate]->(m2:movie) WHERE r.stars>3 AND r2.stars>3 RETURN m, u,m2
```
