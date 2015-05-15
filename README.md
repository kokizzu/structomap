# Go serializer [![Build Status](https://travis-ci.org/tuvistavie/serializer.svg)](https://travis-ci.org/tuvistavie/serializer) [![GoDoc](https://godoc.org/github.com/tuvistavie/serializer?status.svg)](https://godoc.org/github.com/tuvistavie/serializer)

This package helps you to serialize your `struct` into `map` easily. It provides a `serializer.Serializer` interface implemented by the `serializer.Base` type which contains chainable function to add, remove or modify fields. The result is returned as a `map[string]interface{}`.
It is then up to you to encode the result in JSON, XML or whatever you like.

Here is an example.

```go
import "github.com/tuvistavie/serializer"

type User struct {
    ID        int
    Email     string
    HideEmail bool
    FirstName string
    LastName  string
    CreatedAt time.Time
    UpdatedAt time.Time
}

currentTime := time.Date(2015, 05, 13, 15, 30, 0, 0, time.UTC),

user := User{
    ID: 1, Email: "x@example.com", FirstName: "Foo", LastName:  "Bar",
    HideEmail: true, CreatedAt: currentTime, UpdatedAt: currentTime,
}
userMap := serializer.New(user).
              UseSnakeCase().
              Pick("ID", "FirstName", "LastName", "Email").
              PickFunc(func(t interface{}) interface{} {
                  return t.(time.Time).Format(time.RFC3339)
              }, "CreatedAt", "UpdatedAt").
              OmitIf(func(u interface{}) bool {
                  return u.(User).HideEmail
              }, "Email").
              Add("CurrentTime", time.Date(2015, 5, 15, 17, 41, 0, 0, time.UTC)).
              AddFunc("FullName", func(u interface{}) interface{} {
                  return u.(User).FirstName + " " + u.(User).LastName
              }).Result()
str, _ := json.MarshalIndent(userMap, "", "  ")
fmt.Println(string(str))
```

will give:

```json
{
  "created_at": "2015-05-13T15:30:00Z",
  "current_time": "2015-05-15T17:41:00Z",
  "first_name": "Foo",
  "full_name": "Foo Bar",
  "id": 1,
  "last_name": "Bar",
  "updated_at": "2015-05-13T15:30:00Z"
}
```

The full documentation is available at https://godoc.org/github.com/tuvistavie/serializer.

## Choosing a key format

You can set the key format for the output map using `UseSnakeCase()`, `UsePascalCase()` or `UseCamelCase()` on the serializer object.
You can also set the default case for all new serializers by using
`serializer.SetDefaultCase(serializer.SnakeCase)` (`serializer.CamelCase` and `serializer.PascalCase` are also available). The `init()` function would be a good place to set this.

## Building your own serializer

With `Serializer` as a base, you can easily build your serializer.

```go
type UserSerializer struct {
  *serializer.Base
}

func NewUserSerializer(user User) *UserSerializer {
  u := &UserSerializer{serializer.New(user)}
  u.Pick("ID", "CreatedAt", "UpdatedAt", "DeletedAt")
  return u
}

func (u *UserSerializer) WithPrivateInfo() *UserSerializer {
  u.Pick("Email")
  return u
}

userMap := NewUserSerializer(user).WithPrivateInfo().Result()
```

Note that the `u.Pick`, and all other methods do modify the serializer, they do not return a new serializer each time. This is why it works
even when ignoring `u.Pick` return value.

## License

This is released under the MIT license. See the [LICENSE](./LICENSE) file for more information.
