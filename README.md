# gormgen

gormgen is a code generation tool to generate code that's compatible with the [gorm ORM](https://github.com/jinzhu/gorm) but without having to deal with `interface{}`s or with database column names.

## Why to use gormgen

```go

// Querying

// The gorm way:
users := []User{}
err := db.Where("age > ?", 20).Order("age ASC").Limit(10).Find(&users).Error

// gormgen way
users, err := (&UserQueryBuilder{}).
  WhereAge(gormgen.GreaterThanPredict, 20).
  OrderByAge(true).
  Limit(10).
  QueryAll(db)


// Creating Object
user := &User{
  Name: "Bla",
  Age: 20,
}

// The gorm way
db.Create(user)

// The gormgen way
user.Save(db)
```

- No more ugly `interface{}`s when doing in the `Where` function. Using gormgen, the passed values will be type checked.
- No more ugly strings for column names for `Where` and `Order` functions. By this, you won't need to convert the field name to the column name yourself, gormgen will do it for you. Also, you won't forget to change a column name when you change the field name because your code won't compile until you fix it everywhere.
- A more intuitive way to return the results instead of passing them as a param.
- The generated code is compatible with gorm. So you can use other gorm functions that are not yet implemented in gormgen normally.

## How it works

If you have the following :

```go
//go:generate gormgen -structs User -output user_gen.go
type User struct {
	ID   uint   `gorm:"primary_key"`
	Name string
	Age  int
}
```

gormgen will generate for you :

```go
func (t *User) Save(db *gorm.DB) error {/* … */}
func (t *User) Delete(db *gorm.DB) error {/* … */}
type UserQueryBuilder struct {/* … */}
func (qb *UserQueryBuilder) Count(db *gorm.DB) (int, error) {/* … */}
func (qb *UserQueryBuilder) First(db *gorm.DB) (*User, error) {/* … */} // Sorted by primary key
func (qb *UserQueryBuilder) QueryOne(db *gorm.DB) (*User, error) {/* … */} // Sorted by the order specified
func (qb *UserQueryBuilder) QueryAll(db *gorm.DB) ([]User, error) {/* … */}
func (qb *UserQueryBuilder) Limit(limit int) *UserQueryBuilder {/* … */}
func (qb *UserQueryBuilder) Offset(offset int) *UserQueryBuilder {/* … */}
func (qb *UserQueryBuilder) WhereID(p gormgen.Predict, value uint) *UserQueryBuilder {/* … */}
func (qb *UserQueryBuilder) OrderByID(asc bool) *UserQueryBuilder {/* … */}
func (qb *UserQueryBuilder) WhereName(p gormgen.Predict, value string) *UserQueryBuilder {/* … */}
func (qb *UserQueryBuilder) OrderByName(asc bool) *UserQueryBuilder {/* … */}
func (qb *UserQueryBuilder) WhereAge(p gormgen.Predict, value int) *UserQueryBuilder {/* … */}
func (qb *UserQueryBuilder) OrderByAge(asc bool) *UserQueryBuilder {/* … */}
```

## How to use it

- `go get -u github.com/MohamedBassem/gormgen`
- Add the `//go:generate` comment mentioned above anywhere in your code.
- Add `go generate` to your build steps.
- **The generated code will depend on gorm and gormgen, so make sure to vendor both of them.**

## Not yet supported features

- [X] Inferring database column name from gorm convention or gorm struct tag.
- [X] Ignoring fields with `gorm:"-"`.
- [ ] Support for anonymous structs (IMPORTANT for gorm.Model).
- [ ] Support for type aliases.
- [ ] Support for detecting and querying with primary key.

## Contributing

Your contributions and ideas are welcomed through issues and pull requests.

*Note for development* : Make sure to have `gormgen` in your path to be able to run the tests. Also, always run the tests with `make test` to regenerate the test structs.