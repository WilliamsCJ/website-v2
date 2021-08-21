+++
date = 2021-08-14T11:00:00Z
draft = true
linktitle = "StudentLayer Part 5 - Making our service useful"
series = ["StudentLayer"]
title = "StudentLayer Part 5 - Making our service useful"
type = ["post", "post"]
weight = 10
[author]
name = "CJ Williams"

+++
In the previous post, we cleaned up our dataset and refined the data model. Now it's time to implement this model and utilise it in our service.

> The code for this post can be found [here](https://github.com/WilliamsCJ/studentlayer/commit/116f14ab5aed689863cfe52fef7aeaaf42f6d0ad)

## Creating our models

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

From this, we can create a corresponding model in Go using a _struct_:

```go
type University struct {
	Name         string   `json:"name"`
	Domains      []string `json:"domain"`
	AlphaTwoCode []string `json:"alpha_two_code"`
}
```

Note the use of the JSON tags in our struct - these will tell the _encoding/json_ package how to unmarshal a string of JSON into our struct.

Now, remember that our _universities_clean.json_ file is an array of JSON objects. Therefore we will find it useful also to define another type that is a slice of University objects:

```go
type UniversityCollection []University
```

Lastly, we also want to create a model to represent an email address that we can verify. Let's try something like this:

```go
type Email struct {
	Email        string               `json:"email"`
	Verified     bool                 `json:"verified"`
	Universities UniversityCollection `json:"universities,omitempty"`
}
```

At a minimum, we need the string field to hold our email address and a boolean field to denote whether the email has been verified. Still, it could also be useful to include the returned universities when we verify our email. This information could be useful to a user of the service, perhaps for ensuring they only verify students in a particular country. Notice, however, that our Universities field is of the type _UniversitiesCollection_ - this is because we know (from our previous post) that multiple universities could use a domain.

As previously, we've added JSON tags to our struct. This will be useful for marshalling/unmarshalling. Specifically, using the omitempty tag will allow us not to return the Universities field if it is empty (i.e. the email is unverified).

We'll put these in a file called _models.go_

## Loading our dataset

Now that we've defined our models, we need to store our dataset in a place that our program can access. To begin, we will store this data in memory using a _map_, specifically a `map[string]UniversitiesCollection`. This will allow us to search for universities by domain. For reasons that will become apparent later, we will store our map in a _struct_ called server like so:

```go
type Service struct {
	Store  map[string]UniversityCollection
}
```

We can now create a `Service` in our `main` function as so:

```go
router := http.NewServeMux() // This is unchanged from before
	
server := Service{
	Store: make(map[string]UniversityCollection),
}
```

Notice we call the `make` function when we initialise our `Service` - if we don't `Service.Store` will be a `nil` map, which will cause errors when we try to access it.

Now we can create a method to load our data into the `Service.Store`:

```go
func (s *Service) LoadUniversities(filepath string) {
	// Open file.
	file, err := os.Open(filepath)
	if err != nil {
		panic(err)
	}

	// Defer closing the file, and check for error during close.
	defer func() {
		err = file.Close()
		if err != nil {
			panic(err)
		}
	}()

	// Read bytes in file.
	bytes, err := ioutil.ReadAll(file)
	if err != nil {
		panic(err)
	}

	// Unmarshal the json into a UniversityCollection struct.
	var universities UniversityCollection
	err = json.Unmarshal(bytes, &universities)
	if err != nil {
		panic(err)
	}

	// For each university, add each domain to the key/value store (a map) where the key is the domain.
	for _, university := range universities {
		for _, domain := range university.Domains {
			existing, exists := s.Store[domain]

			// If the domain is already in the store, append the domain to the UniversityCollection.
			// Else create a new UniversityCollection with just the university to start with.
			if exists {
				s.Store[domain] = append(existing, university)
			} else {
				s.Store[domain] = UniversityCollection{university}
			}
		}
	}
}
```

This function opens and reads our file before attempting to unmarshal the JSON data into a `UniversitiesCollection`. Once this is done, we iterate through each domain of each university. If the domain is present in the map, we append it to the `UniversitiesCollection` referenced by that domain. If not, we create a new `UniversitiesCollection` containing the university.

For some, the function signature may not be familiar. In fact, this is not a function - it is a **method**! In Go, methods are functions that include a receiver, `(*s Service)` in our case. This means that our function is attached to the type defined in the receiver, `Service`, and defines the behaviour of the type.

In practice, we can call the method like so, to load our universities data into `Service.Store`:

```go
router := http.NewServeMux()

server := Service{
	Store: make(map[string]UniversityCollection),
}

server.LoadUniversities("data/universities_clean.json")
```

## Creating a new route

In part 2, we created a route that our router would serve. This was only a basic Hello World route, and we now need to create a route that users of our service can POST email addresses to:

```go
router := http.NewServeMux()

server := Service{
	Store: make(map[string]UniversityCollection),
}

server.LoadUniversities("data/universities_clean.json")

router.HandleFunc("/emails", server.PostEmail)
```

This is similar to how we defined a route previously. However, instead of using an anonymous function, we've used another method of our `Service` struct. This will allow our handler `PostEmail` to access the store. We can define `PostEmail` as such:

```go
func (s *Service) PostEmail(w http.ResponseWriter, r *http.Request) {
	log.Printf("Serving %s request for %s", r.Method, r.URL)

	// Check that the request method is POST
	if r.Method != http.MethodPost {
		// Set status code to 405 MethodNotAllowed
		w.WriteHeader(http.StatusMethodNotAllowed)

		// Write error in response body
		resp := []byte("405 - Method not allowed")
		bytes, err := w.Write(resp)
		if err != nil {
			panic(err)
		}
		if bytes != len(resp) {
			panic(err)
		}
	}

	// Decode JSON from body into Email struct
	var email Email
	err := json.NewDecoder(r.Body).Decode(&email)
	if err != nil {
		// Set status code to 400 Bad Request
		w.WriteHeader(http.StatusBadRequest)
		resp := []byte("500 - Internal Server Error")

		// Write error in response body
		bytes, err := w.Write(resp)
		if err != nil {
			panic(err)
		}
		if bytes != len(resp) {
			panic(err)
		}
	}

	// Check the store for the university, using two-value assignment for the university and boolean result
	emailParts := strings.Split(email.Email, "@")
	if len(emailParts) != 2 {
		// Set status code to 400 Bad Request
		w.WriteHeader(http.StatusBadRequest)
		resp := []byte("500 - Internal Server Error")

		// Write error in response body
		bytes, err := w.Write(resp)
		if err != nil {
			panic(err)
		}
		if bytes != len(resp) {
			panic(err)
		}
	}

	universities, verified := s.Store[emailParts[1]]
	email.Universities = universities
	email.Verified = verified

	// Write the email struct in body response
	bytes, err := json.Marshal(email)
	_, err = w.Write(bytes)
	if err != nil {
		panic(err)
	}
}
```

In our new handler, we check that we have a POST request before reading the body and unmarshal the (hopefully) JSON into an `Email` struct. Then we search for the domain of the email and then return the result.

And that's it! We can tie everything up by making sure that we run our server in the `main` function:

```go
func main() {
	router := http.NewServeMux()

	server := Service{
		Store: make(map[string]UniversityCollection),
	}

	server.LoadUniversities("data/universities_clean.json")

	router.HandleFunc("/emails", server.PostEmail)

	log.Println("Running on localhost:8080...")
	log.Fatal(http.ListenAndServe(":8080", router))
}
```

## Wrapping up

We now have a useful service that can be used to (sort of) verify emails. Amazing! However, we are far from done. There are plenty more features to add, and in the next few posts, we'll look at refactoring and cleaning up our code.

Thanks for reading!