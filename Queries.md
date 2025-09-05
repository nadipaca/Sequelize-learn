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

### Creating in bulk
1. Method to create multiple records at once, with only one query
```
const captains = await Captain.bulkCreate([{name: 'jack Sparrow' }, {name: 'Davy Jones' }]);
```

### Ordering and Grouping
1. Use order and group array which acts like ORDER BY and GROUP BY. Order option takes an array of items to order the query.
2.  Customize ordering by
```
Users.findAll(
order: [
[`title`,`DESC`],
[sequelize.fn(`max',sequelize.col('age'))], // will order by max age
]
)
```
3. Order Users by their tasks created At
```
// A User has many Tasks
const User = sequelize.define('User', {
  name: DataTypes.STRING,
});

// A Task belongs to a User and has many Subtasks
const Task = sequelize.define('Task', {
  title: DataTypes.STRING,
});

// A Task belongs to a Project
const Project = sequelize.define('Project', {
  name: DataTypes.STRING,
});

// A Subtask belongs to a Task
const Subtask = sequelize.define('Subtask', {
  name: DataTypes.STRING,
});

// Associations
User.hasMany(Task);
Task.belongsTo(User);

Task.belongsTo(Project);
Project.hasMany(Task);

Task.hasMany(Subtask);
Subtask.belongsTo(Task);
```
--- Query ---
```
(SELECT "Users".* 
FROM "Users"
LEFT JOIN "Tasks" ON "Users"."id" = "Tasks"."UserId"
ORDER BY "Tasks"."createdAt" DESC;)

const users = await User.findAll({
  include: [Task],
  order: [[Task, 'createdAt', 'DESC']]
});
```

4. Order Users by their Tasks’ Project createdAt
```
const users = await User.findAll({
  include: [{ model: Task, include: [Project] }],
  order: [[Task, Project, 'createdAt', 'DESC']]
});
```
```
SELECT "Users".* 
FROM "Users"
LEFT JOIN "Tasks" ON "Users"."id" = "Tasks"."UserId"
LEFT JOIN "Projects" ON "Tasks"."ProjectId" = "Projects"."id"
ORDER BY "Projects"."createdAt" DESC;
```

5. Order Subtasks by their Task Created At
```
const subtasks = await Subtask.findAll({
  include: [Task],
  order: [[Subtask.associations.Task, 'createdAt', 'DESC']]
});
```
```
SELECT "Subtasks".* 
FROM "Subtasks"
LEFT JOIN "Tasks" ON "Subtasks"."TaskId" = "Tasks"."id"
ORDER BY "Tasks"."createdAt" DESC;
```

6. Order Users by their Task's project CreatedAt
   ```
   const users = await Users.findAll({
   include: [{model: Task, include: [Project]}],
   order: [[Task,Project,'CreatedAt','DESC']]
   })
   ```
   ```
   SELECT "Users".*
   FROM "Users"
   LEFT JOIN "Taks" ON "Users"."id" = "Tasks"."userId"
   LEFT JOIN "Projects" ON "Projects.TaskId" = "Project.id"
   ORDER BY "Projects"."createdAt" DESC;

7. Order Subtasks by their Task's CreatedAt
   ```
   const subtasks = await Subtask.findAll({
   include: [Task],
   order:[[SubTask.associations.Task,'createdAt','DESC']]
   });
   ```
   ```
   SELECT "Subtasks".*
   FROM "Subtasks"
   LEFT JOIN "TASKS" ON "Subtasks"."TaskId" = "Tasks"."id"
   ORDER BY "TASKS"."CreatedAt" DESC;
   
### Grouping
```
project.findAll({group: 'name' });

### Limits and Pagination

```
// fetch 10 instances/rows
Project.findAll({ limit: 10 });

// Skip 8 instances / rows
Project.findAll({ offset: 8 })

// Skip 5 instances and fetch the 5 after that
Project.findAll({offset: 5, limit: 5 })
