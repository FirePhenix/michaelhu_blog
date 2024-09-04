+++
title = "Let's make your slice/map easier to maintain"
date = 2024-08-28T16:57:21-04:00
draft = true
tags = ["Go", "Dev"]
ShowToc = true
TocOpen = true
+++

### Introduction
As a Go and Ruby on Rails (RoR) developer, I often find myself building microservices that interact with RoR applications. One of the most annoying challenges is maintaining simple data structures like maps or slices, especially when they need to evolve. For example, let's take a look at how we define a slice of Partner structs:

``` go
type Partner struct {
    Name string
}

func ListPartners() []Partner {
    return []Partner{
        {Name: "partner1"},
        {Name: "partner2"},
        {Name: "partner3"},
        {Name: "partner4"},
    }
}

    
```
At first glance, this looks fine. But what happens when your PM asks you to return only some of these names based on certain conditions? You’ll need to modify ListPartners to filter by name. Here’s one possible solution:

``` go
func ListPartners(name string) []Partner {
    partners := []Partner{
        {Name: "partner1"},
        {Name: "partner2"},
        {Name: "partner3"},
        {Name: "partner4"},
    }

    if name == "" {
        return partners
    }

    filtered := make([]Partner, 0)

    for _, partner := range partners {
        if strings.Contains(partner.Name, name) {
            filtered = append(filtered, partner)
        }
    }

    return filtered
}

```
Now, you can filter partners by name:

```go
func main() {
    partners := ListPartners("partner1")
    fmt.Printf("partners %+v\n", partners)
}

```

This works fine, but what happens when your PM wants more filtering options—such as checking the status or filtering by creation date? The function can quickly become bloated and hard to maintain. Let’s look at a better, more maintainable approach.

### Refactor with Struct Methods
We can make the code cleaner and easier to extend by using a custom type and adding methods for filtering. Let's start by introducing a new type:
```go
type Partners []Partner

func ListPartner() Partners {
    return Partners{
        {Name: "partner1"},
        {Name: "partner2"},
        {Name: "partner3"},
        {Name: "partner4"},
    }
}

```
Now, we can add a method to filter by name:

```go
func (p Partners) FilterByName(name string) Partners {
    filtered := make(Partners, 0)

    for _, partner := range p {
        if strings.Contains(partner.Name, name) {
            filtered = append(filtered, partner)
        }
    }

    return filtered
}

```
And if you need additional methods, you can easily add them:

```go
func (p Partners) CheckStatus() Partners {
    // Add logic to filter or check status
    return p
}

func (p Partners) AddRecord(partner Partner) Partners {
    return append(p, partner)
}

func (p Partners) Len() int {
    return len(p)
}

```
### Conclusion
By introducing the Partners type and adding methods, your code becomes more maintainable and easier to extend. You can now add new filtering conditions or logic without modifying the core ListPartners function every time, keeping your code clean and flexible.