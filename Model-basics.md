Promises and async/await

Most of the methods provided by sequelize are asynchronous and therefore return promises, They are all promises, so we can use Promise API for example using then, catch, and finally. we can aslo async and await.

## Models
1. Model is an abstraction that represent a table in DB. In sequelize that is a class that extends **Model**. 
2. The model tells sequelize several things about entity that is table in DB and which columns it has and their datatypes.
3. A model in seq has a name, does not have to be same as Table name. Usually models have singular names such as **USer**. While tables have Pluralized names such as Users.

## Model Definition
1. Models can be defined in two equivalent ways
   a. Calling sequelize.define(modelName, attributes, options)
   b. Extending Model and calling init(attributes, options)
2. After a model is defined, it is available in **sequelize.models** by its model's name.

### Using Sequelize.define:
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

const User = sequelize.define(
 'User',
 {
 //Model attributes are defined here
 
 firstName: {
 type: DataTypes.STRING,
 allowNull: false,
 },
 
 lastName: {
 type: DataTypes.STRING,
 //allowNull defaults ti true
 }

 ...other options
)

`sequelize.define` also returns the model
console.log(User == sequelize.models.User);  //true 

### Extending Model
const { Sequelize, DataTypes, Model} = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

class user extends Model {}
User.init(
//Model attributes are defined here
 
 firstName: {
 type: DataTypes.STRING,
 allowNull: false,
 },
 
 lastName: {
 type: DataTypes.STRING,
 //allowNull defaults ti true
 }
{
 ...other options
 sequelize,
 modelName: 'User'
 }
)


By default, when the table name is not given, Sequelize automatically pluralizes the model name and uses that as the table name. Using a library called inflection.

## Providing table name
sequelize.define(
'USer'
{
// ...attributes
},
{
tableName: 'Employees',
}
);

## Model Synchronization
A model can be synchronize with the databse by calling **model.sync(options)**, it will syncs automically if a table doesn't exist on DB, or if it has different or less columns or any other diff...

User.sync() -> Creates table if it doesn't exisr (does nothing if already exists)
User.sync({force: true}) - creates table, dropping if already exists
User.sync({alter:true}) -> This checks what is the current state of the table in the database (which columns it has, what are their data types, etc), and then performs the necessary changes in the table to make it match the model.

## Sync all models at once 
await sequelize.sync({force: true});

## Dropping all tables
await User.drop();

## Drop all tables
await sequelize.drop();

## Timestamps
1. By default sequelize automatically adds the fields createdAt, updatedAt to every model, using datatype DataTypes.DATE,. 
2. This behavior can be disabled for a model with timeStamps:false option.
3. Enable only one
   class Foo extends Model {}
   Foo.init(
   { /* attributes */ }
   {
      sequelize,
      timestamps: true,
      createdAt: false,
      updatedAt: 'updateTimestamp'
   }
   )

## Default values
sequelize.define('User', {
  name: {
    type: DataTypes.STRING,
    defaultValue: 'John Doe',
  },
  createdAt: {
  type: DataTypes.DATETIME,
    defaultValue: DataTypes.NOW,
  }
});

## DataTypes 
STRING, INTEGER, DATE, UUID, REAL, DOUBLE
