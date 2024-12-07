# Blox - Assessment
`Credit: Anjali Taksali`

## Q1.
Elaborate what your internship or academic projects were?

### Project: URL Shortening

#### a. What did the system do?
> The URL shortening service allowed users to convert long URLs into shortened versions. The system handled user input for URL shortening and ensured efficient redirection using a unique short identifier. It was implemented with the goal of optimizing network traffic and providing a clean, easily shareable link.
 
#### b. What other systems have you seen in the wild like that?
> Popular systems such as Bit.ly, TinyURL, and Google's URL shortener (which has now been deprecated) are similar to this service. These platforms provide users with short links that redirect to the original long URL, often including features like analytics and customizable aliases.
 
#### c. How do you approach the development problem?
>   To solve the URL shortening problem, I followed an iterative approach;
>  - Start with basic URL storage and retrieval functionality.
>  - Use a hashing mechanism to create unique shortened URLs.
>  - Ensure fast access with the use of a database (in my case, DynamoDB).
>  - Add basic analytics for tracking how often the shortened URL was accessed.

 
#### d. What were interesting aspects where you copied code from Stack Overflow?
>  - For generating unique short URLs, I used a base-64 encoding technique to convert integers into alphanumeric strings. I found several efficient implementations of this on Stack Overflow.  
>  - I also referenced solutions on how to handle HTTP redirects efficiently to ensure minimal latency when accessing shortened links.

 
#### e. What did you learn from some very specific copy paste? Mention explicitly some of them.
>  - **Base-64 encoding**: I had to look up optimized versions of base-64 encoding since I initially wrote my own which didn’t scale well. By examining examples on Stack Overflow, I found that most of the solutions used efficient algorithms that helped me minimize the size of the generated shortened URL and increase the speed of access.  
>  - **Handling 301 Redirects**: There were a lot of common issues with redirecting users quickly and correctly. By copying a redirect snippet from Stack Overflow, I learned how HTTP response codes (such as 301) should be used properly for permanent redirects, ensuring that search engines and clients can cache and route requests effectively.
 

## Q2.
There is one database. Let’s say it is hosted locally and one of the team members migrates it to AWS or GCP. How can one confirm that the copied data is the same as the original data? What would be the check points?

### DB Migration `( verify_data.go )`

```go
package main

import (
	"fmt"
	"log"
	"reflect"
)

// Function to simulate fetching data from a database (replace with actual DB queries)
func fetchDataFromDB(dbType string) ([]map[string]string, error) {
	// Replace with actual DB querying logic.
	// This is just dummy data for illustration.
	if dbType == "local" {
		return []map[string]string{
			{"ID": "1", "Name": "John", "Age": "30"},
			{"ID": "2", "Name": "Alice", "Age": "25"},
		}, nil
	} else if dbType == "migrated" {
		return []map[string]string{
			{"ID": "1", "Name": "John", "Age": "30"},
			{"ID": "2", "Name": "Alice", "Age": "25"},
		}, nil
	}
	return nil, fmt.Errorf("Unknown DB type")
}

// Function to compare two slices of maps
func compareData(localData, migratedData []map[string]string) bool {
	return reflect.DeepEqual(localData, migratedData)
}

func main() {
	// Fetch data from both databases (local and migrated)
	localData, err := fetchDataFromDB("local")
	if err != nil {
		log.Fatalf("Error fetching local data: %v", err)
	}

	migratedData, err := fetchDataFromDB("migrated")
	if err != nil {
		log.Fatalf("Error fetching migrated data: %v", err)
	}

	// Compare the data
	if compareData(localData, migratedData) {
		fmt.Println("The data matches!")
	} else {
		fmt.Println("The data does NOT match!")
	}
}
```

## Q3.
Write a function to parse any valid json string into a corresponding Object, List, or Map
object. Note that the integer and floating point should be arbitrary precision.

### DB Migration `( parse_json.go )`

```go
package main

import (
	"encoding/json"
	"errors"
	"fmt"
	"math/big"
	"strings"
)

// Custom type for handling JSON numbers with arbitrary precision
type ArbitraryNumber struct {
	Value interface{}
}

// UnmarshalJSON implements the json.Unmarshaler interface for ArbitraryNumber
func (n *ArbitraryNumber) UnmarshalJSON(data []byte) error {
	str := strings.TrimSpace(string(data))
	// Check for integers
	if intVal, ok := new(big.Int).SetString(str, 10); ok {
		n.Value = intVal
		return nil
	}
	// Check for floating-point numbers
	if floatVal, _, err := big.ParseFloat(str, 10, 0, big.ToNearestEven); err == nil {
		n.Value = floatVal
		return nil
	}
	return errors.New("invalid number format")
}

// ParseJSON parses a JSON string into Go types while handling arbitrary precision
func ParseJSON(input string) (interface{}, error) {
	// Custom decoder to parse JSON numbers as ArbitraryNumber
	var parse func(interface{}) (interface{}, error)
	parse = func(data interface{}) (interface{}, error) {
		switch v := data.(type) {
		case map[string]interface{}:
			// Recursively parse map values
			parsedMap := make(map[string]interface{})
			for key, value := range v {
				parsedValue, err := parse(value)
				if err != nil {
					return nil, err
				}
				parsedMap[key] = parsedValue
			}
			return parsedMap, nil
		case []interface{}:
			// Recursively parse list elements
			parsedList := make([]interface{}, len(v))
			for i, value := range v {
				parsedValue, err := parse(value)
				if err != nil {
					return nil, err
				}
				parsedList[i] = parsedValue
			}
			return parsedList, nil
		case float64:
			// Handle numbers
			num := ArbitraryNumber{}
			if err := json.Unmarshal([]byte(fmt.Sprintf("%v", v)), &num); err != nil {
				return nil, err
			}
			return num.Value, nil
		default:
			// Other data types (strings, booleans, nil)
			return v, nil
		}
	}

	// Decode raw JSON into a generic interface{}
	var rawData interface{}
	if err := json.Unmarshal([]byte(input), &rawData); err != nil {
		return nil, err
	}

	return parse(rawData)
}

// Example usage
func main() {
	jsonStr := `{
		"int": 123456789012345678901234567890,
		"float": 12345.678901234567890123456789,
		"list": [1, 2.3, 4567890123456789012345],
		"string": "Hello, World!",
		"boolean": true,
		"null": null
	}`

	parsedData, err := ParseJSON(jsonStr)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}

	fmt.Printf("Parsed Data: %+v\n", parsedData)
}

```

