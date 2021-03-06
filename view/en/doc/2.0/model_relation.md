## Relational Model

Table in database often related to other tables and need to be operated with related tables. For example, an article can have category, tag, comment and author.

ThinkJS supports relational model which can simple these operations.

### Supported Type

Relational model supports four usual relationship:

- `think.model.HAS_ONE` one to one model
- `think.model.BELONG_TO` one to one belong to
- `think.model.HAS_MANY` one to many
- `think.model.MANY_TO_MANY` many to many

### Create Relational Model

Use `thinkjs model [name] --relation` to create relational model:

```js
thinkjs model home/post --relation
```

This will create model file `src/home/model/post.js`.

### Set Relationship

Use `relation` property to set relationship:

```js
export default class extends think.model.relation {
  init(...args){
    super.init(...args);
    //use relation property to set relationship, can set many relationships
    this.relation = {
      cate: {},
      comment: {} 
    }
  }
}
```

You can also use ES7 syntax to define `relation` property:

```js
export default class extends think.model.relation {

  //define relation property directly
  relation = {
    cate: {},
    comment: {} 
  }

  init(...args){
    super.init(...args);
  }
}
```

#### Data Format of Single Relational Model

```js
export default class extends think.model.relation {
  init(...args){
    super.init(...args);
    this.relation = {
      cate: {
        type: think.model.MANY_TO_MANY, //relation type
        model: "", //model name
        name: "profile", //data name
        key: "id", 
        fKey: "user_id", //forign key
        field: "id,name",
        where: "name=xx",
        order: "",
        limit: "",
        rModel: "",
        rfKey: ""
      },
    }
  }
}
```

Each fields means:

- `type` type of relation
- `model` model name of relation table, default is key, here is `cate`
- `name` data field name, default is key, here is `cate`
- `key` related key of current model
- `fKey` related key of related table
- `field` field used to query related table, fKey must be included if you set this field
- `where` where condition used to query related table
- `order` order used to query related table
- `limit` limit used to query related table
- `page` page used to query related table
- `rModel` related model name in many to many type
- `rfKey` key in related table in many to many type

If you just want to set related type without other fields, you can use this simple way:

```js
export default class extends think.model.relation {
  init(...args){
    super.init(...args);
    this.relation = {
      cate: think.model.MANY_TO_MANY
    }
  }
}
```

#### HAS_ONE

One to one relation, means current table has one additional table.

Suppose curret model name is `user` and related table model name is `info`, then `key` field in config default is `id`, `fKey` default is `user_id`.

```js
export default class extends think.model.relation {
  init(..args){
    super.init(...args);
    this.relation = {
      info: think.model.HAS_ONE
    }
  }
}
```

Execute quering operation will get below data:

```js
[
  {
    id: 1,
    name: "111",
    info: { //关联表里的数据信息
      user_id: 1,
      desc: "info"
    }
  }, ...]
```

#### BELONG_TO

One to one relation, means belong to one table, reverse of HAS_ONE.

Suppose current model name is `info`, related table model name is `user`, then `key` field in config default is `user_id` and `fKey` default is `id`.

```js
export default class extends think.model.relation {
  init(..args){
    super.init(...args);
    this.relation = {
      user: think.model.BELONG_TO
    }
  }
}
```

Execute quering operation will get below data:

```js
[
  {
    id: 1,
    user_id: 1,
    desc: "info",
    user: {
      name: "thinkjs"
    }
  }, ...
]
```

#### HAS_MANY

One to many relation.

Suppose current model name is `post`, related table model name is `comment`, then `key` field in config default is `id` and `fKey` default is `post_id`.

```js
"use strict";
/**
 * relation model
 */
export default class extends think.model.relation {
  init(...args){
    super.init(...args);

    this.relation = {
      comment: {
        type: think.model.HAS_MANY
      }
    }
  }
}
```

Execute quering operation will get below data:

```js
[{
  id: 1,
  title: "first post",
  content: "content",
  comment: [{
    id: 1,
    post_id: 1,
    name: "welefen",
    content: "first comment"
  }, ...]
}, ...]
```

If data in related table needs pagination, use `page` parameter:

```js
"use strict";
/**
 * relation model
 */
export default class extends think.model.relation {
  init(...args){
    super.init(...args);

    this.relation = {
      comment: {
        type: think.model.HAS_MANY
      }
    }
  }
  getList(page){
    return this.setRelation("comment", {page: page}).select();
  }
}
```

Besides using `setRelation`, you can also pass in a function, this function will be executed during paramater mergin.

#### MANY_TO_MANY

Many to many relation.

Suppose current model name is `post`, related table model name is `cate`, then we need a relationship table.`rModel` field in config default is `post_cate` and `rfKey` default is `cate_id`.

```js
"use strict";
/**
 * relation model
 */
export default class extends think.model.relation {
  init(...args){
    super.init(...args);

    this.relation = {
      cate: {
        type: think.model.MANY_TO_MANY,
        rModel: "post_cate",
        rfKey: "cate_id"
      }
    }
  }
}
```

Quering results will be:

```js
[{
  id: 1,
  title: "first post",
  cate: [{
    id: 1,
    name: "cate1",
    post_id: 1
  }, ...]
}, ...]
```

#### Dead Cycle

Suppose we have two tables, one set the other as HAS_ONE and the other set this as BELONG_TO, this will cause cycle quering during quering and result to dead cycle.

You can set `relation` field in config to close related quering and prevent dead cycle:

```js
export default class extends think.model.relation {
  init(..args){
    super.init(...args);
    this.relation = {
      user: {
        type: think.model.BELONG_TO,
        relation: false //close related quering when query user
      }
    }
  }
}
```

You can also only close current model's relationship:

```js
export default class extends think.model.relation {
  init(..args){
    super.init(...args);
    this.relation = {
      user: {
        type: think.model.BELONG_TO,
        relation: "info" //close info model's relationship whey query user
      }
    }
  }
}
```

### Close Relationship Temporarily

After set relationship, operations like query will query related table automatically. If you don't want to query related table, just use `setRelation` method to close relationship temporarily.

#### Close All

Use `setRelation(false)` to close all relationship query.

```js
export default class extends think.model.relation {
  init(...args){
    super.init(...args);
    this.relation = {
      comment: think.model.HAS_MANY,
      cate: think.model.MANY_TO_MANY
    }
  },
  getList(){
    return this.setRelation(false).select();
  }
}
```

#### Open Part

Use `setRelation('comment')` to query data from `comment`, other table won't be queied.

```js
export default class extends think.model.relation {
  init(...args){
    super.init(...args);
    this.relation = {
      comment: think.model.HAS_MANY,
      cate: think.model.MANY_TO_MANY
    }
  },
  getList2(){
    return this.setRelation("comment").select();
  }
}
```

#### Close Part

Use `setRelation('comment', false)` to close `comment` quering.

```js
export default class extends think.model.relation {
  init(...args){
    super.init(...args);
    this.relation = {
      comment: think.model.HAS_MANY,
      cate: think.model.MANY_TO_MANY
    }
  },
  getList2(){
    return this.setRelation("comment", false).select();
  }
}
```

#### Reopen All

Use `setRelation(true)` to reopen all related quering.

```js
export default class extends think.model.relation {
  init(...args){
    super.init(...args);
    this.relation = {
      comment: think.model.HAS_MANY,
      cate: think.model.MANY_TO_MANY
    }
  },
  getList2(){
    return this.setRelation(true).select();
  }
}
```

### mongo Relational Model

This relational model doesn't work for mongo model, mongo relational model stays here [https://docs.mongodb.org/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/](https://docs.mongodb.org/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/).

This doc stays at [https://github.com/75team/www.thinkjs.org/tree/master/view/zh-CN/doc/2.0/model_relation.md](https://github.com/75team/www.thinkjs.org/tree/master/view/zh-CN/doc/2.0/model_relation.md).