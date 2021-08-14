+++
date = 2021-08-14T11:00:00Z
draft = true
linktitle = "StudentLayer Part 5 - Making our service useful"
series = ["StudentLayer"]
title = "StudentLayer Part 5 - Making our service useful"
type = ["post", "post"]
weight = 10
[author]
name = ""

+++
In the previous post we cleaned up our dataset and refined the data model. Now it's time to implement this model and utilise it in our service. 

# Creating our models

If we cast our minds back to our previous post, you might recall that we ended up with JSON objects that roughly looked like so:

```json
{
    "name": "University of Oxford",
    "domain": [
      "ox.ac.uk",
      "oxford.ac.uk"
    ],
    "alpha_two_code": [
      "GB"
    ]
  }
```

From this, we can create a corresponding model in Go using a *struct*:

```go
type University struct {
	Name         string   `json:"name"`
	Domains      []string `json:"domain"`
	AlphaTwoCode []string `json:"alpha_two_code"`
}
```

Note the use of the JSON tags in our struct - these will tell the *encoding/json* package as to how to unmarshal a string of JSON into our struct.

Now, remember that our *universities_clean.json* file is an array of JSON objects. Therefore we will find it useful to also define another type that is a slice of University objects:

```go
type UniversityCollection []University
```

Lastly, we also want to create an model to represent an email address that we verify. 

```go
type Email struct {
	Email        string               `json:"email"`
	Verified     bool                 `json:"verified"`
	Universities UniversityCollection `json:"universities,omitempty"`
}
```

We'll put these in a file called *models.go*