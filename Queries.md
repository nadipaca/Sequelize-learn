## Creating Model Instance

To create a sequelize model instance, we have to create using build method.

For eg:
```
const { Sequelize, Model, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

const User = sequelize.define('user', {
  name: DataTypes.TEXT,
  favoriteColor: {
    type: DataTypes.TEXT,
    defaultValue: 'green',
  },
  age: DataTypes.INTEGER,
  cash: DataTypes.INTEGER,
});

const jane = User.build({name: 'Jane'});
```

to save an instance into database jane.save();

Note: Almost all the methods in the sequelize are asynchronous calls. build is one of teh very few exceptions.

### Log instances
Directly logging using console.log(jane) wont wonk

```console.log(JSON.stringify(jane)); or console.log(jane.toJSON()); ```


## Queries
### INSERT queries
1. creating a new user
**const jane = await User.create({firstname: 'Jane', lastName: 'Doe' });**

2. this query will create an instance first by running **User.build()** and saving that instance with **jane.save()**;

3. we can also define which value to set in the create method. It is useful if we are saving data in DB based on the values the user is entering.

### SELECT 
1. Selecting all data from the database using ( SELECT * FROM )
   ``` await User.findAll() ```
2. To select only some specific attributes (SELECT username, age FROM )
```
Model.findAll({
attributes: [`username`,'age`],
})
```
3. Attributes can be renamed using a nested array (SELECT firstName as name, age FROM ...)
```
Model.findAll({
  attributes: [['firstName', 'name'],'age']
})
```
4. If we want to get all the attribute but to perform function/calculation on a column (SELECT id, foo, bar, baz, qux, hats, COUNT(hats) AS n_hats FROM )

```
// This is a tiresome way of getting the number of hats (along with every column)
Model.findAll({
  attributes: [
    'id',
    'foo',
    'bar',
    'baz',
    'qux',
    'hats', // We had to list all attributes...
    [sequelize.fn('COUNT', sequelize.col('hats')), 'n_hats'], // To add the aggregation...
  ],
});

// This is shorter, and less error prone because it still works if you add / remove attributes from your model later
Model.findAll({
  attributes: {
    include: [[sequelize.fn('COUNT', sequelize.col('hats')), 'n_hats']],
  },
});
```

5. We can also exclude few attributes
```
Model.findAll({
  attributes: {exclude: [`baz`]}
})
```
## WHERE clause
1. Finding details where authorId is 2 (SELECT * FROM post WHERE authorId = 2 AND status = 'active';)
```
POST.findAll({
  where: {
     authorId: 2,
     status: 'active',
  }
})

2. Also by using OR operator, we can perform many other operations
```
const { Op } = require("sequelize");
Post.findAll({
  where: {
    [Op.and]: [{ a: 5 }, { b: 6 }],            // (a = 5) AND (b = 6)
    [Op.or]: [{ a: 5 }, { b: 6 }],             // (a = 5) OR (b = 6)
    someAttribute: {
      // Basics
      [Op.eq]: 3,                              // = 3
      [Op.ne]: 20,                             // != 20
      [Op.is]: null,                           // IS NULL
      [Op.not]: true,                          // IS NOT TRUE
      [Op.or]: [5, 6],                         // (someAttribute = 5) OR (someAttribute = 6)

      // Using dialect specific column identifiers (PG in the following example):
      [Op.col]: 'user.organization_id',        // = "user"."organization_id"

      // Number comparisons
      [Op.gt]: 6,                              // > 6
      [Op.gte]: 6,                             // >= 6
      [Op.lt]: 10,                             // < 10
      [Op.lte]: 10,                            // <= 10
      [Op.between]: [6, 10],                   // BETWEEN 6 AND 10
      [Op.notBetween]: [11, 15],               // NOT BETWEEN 11 AND 15

      // Other operators

      [Op.all]: sequelize.literal('SELECT 1'), // > ALL (SELECT 1)

      [Op.in]: [1, 2],                         // IN [1, 2]
      [Op.notIn]: [1, 2],                      // NOT IN [1, 2]

      [Op.like]: '%hat',                       // LIKE '%hat'
      [Op.notLike]: '%hat',                    // NOT LIKE '%hat'
      [Op.startsWith]: 'hat',                  // LIKE 'hat%'
      [Op.endsWith]: 'hat',                    // LIKE '%hat'
      [Op.substring]: 'hat',                   // LIKE '%hat%'
      [Op.iLike]: '%hat',                      // ILIKE '%hat' (case insensitive) (PG only)
      [Op.notILike]: '%hat',                   // NOT ILIKE '%hat'  (PG only)
      [Op.regexp]: '^[h|a|t]',                 // REGEXP/~ '^[h|a|t]' (MySQL/PG only)
      [Op.notRegexp]: '^[h|a|t]',              // NOT REGEXP/!~ '^[h|a|t]' (MySQL/PG only)
      [Op.iRegexp]: '^[h|a|t]',                // ~* '^[h|a|t]' (PG only)
      [Op.notIRegexp]: '^[h|a|t]',             // !~* '^[h|a|t]' (PG only)

      [Op.any]: [2, 3],                        // ANY (ARRAY[2, 3]::INTEGER[]) (PG only)
      [Op.match]: Sequelize.fn('to_tsquery', 'fat & rat') // match text search for strings 'fat' and 'rat' (PG only)

      // In Postgres, Op.like/Op.iLike/Op.notLike can be combined to Op.any:
      [Op.like]: { [Op.any]: ['cat', 'hat'] }  // LIKE ANY (ARRAY['cat', 'hat'])

      // There are more postgres-only range operators, see below
    }
  }
});
```
3. Advanced queries
Post.findAll({
  where: sequelize.where(sequelize.fn('char_length', sequelize.col('content')), 7),
});
// SELECT ... FROM "posts" AS "post" WHERE char_length("content") = 7

### Update queries

1. Update queries by using where operation

```
await User.update(
{lastName: 'Doe'},
{
 where: {
 lastName: null,
 },
},
)

### DELETE queries
1. Destroying specific data based on condition
```
await User.destroy({
  where: {
   firstName: 'John',
 }
})
```
2. To destroy everything use TRUNCATE SQL
   await User.destroy({
   truncate: true,
   });

