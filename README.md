Using go composition to execute a base implementation in addition of a custom one, and deciding on which dependency a component will recieve based on an arbritary flag.

```go
package main

import "fmt"

type Repository interface {
	FindAll() ([]string, error)
}

type SimpleRepository struct{}
type CustomRepository struct {
	base Repository
}

func NewSimpleRepository() *SimpleRepository {
	return &SimpleRepository{}
}
func (*SimpleRepository) FindAll() ([]string, error) {
	return []string{"@", "#"}, nil
}

func NewCustomRepository(base Repository) *CustomRepository {
	return &CustomRepository{base}
}
func (c *CustomRepository) FindAll() ([]string, error) {
	items, err := c.base.FindAll()
	if err != nil {
		return []string{}, err
	}
	return append([]string{"1", "2", "3"}, items...), nil
}

type Component struct {
	repo Repository
}

func (c *Component) All() []string {
	items, err := c.repo.FindAll()
	if err != nil {
		return []string{}
	}
	return items
}

func NewComponent(repo Repository) *Component {
	return &Component{repo}
}

func componentBuilder(flag int) *Component {
	if flag > 0 {
		return NewComponent(NewCustomRepository(NewSimpleRepository()))
	}
	return NewComponent(NewSimpleRepository())
}

func main() {
	flag := 1
	component := componentBuilder(flag)
	fmt.Println(component.All())
}

```
