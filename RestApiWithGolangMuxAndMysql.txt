package main

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"

	_ "github.com/go-sql-driver/mysql"
	"github.com/gorilla/mux"
)

type Persons struct {
	Person_id   int    `json:"person_id"`
	Person_name string `json:"person_name"`
	Person_city string `json :"person_city"`
}

var db *sql.DB
var err error

func main() {
	db, err = sql.Open("mysql", "root:Saurav@123@tcp(127.0.0.1:3306)/my_database")
	if err != nil {
		panic(err.Error())
	}
	defer db.Close()
	router := mux.NewRouter()
	router.HandleFunc("/persons", getPersons).Methods("GET")
	router.HandleFunc("/persons", createPerson).Methods("POST")
	router.HandleFunc("/persons/{pid}", getPerson).Methods("GET")
	router.HandleFunc("/persons/{pid}", updatePerson).Methods("PUT")
	router.HandleFunc("/persons/{pid}", deletePerson).Methods("DELETE")
	http.ListenAndServe(":8000", router)
}
func getPersons(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")

	var persons []Persons
	result, err := db.Query("SELECT person_id, person_name, person_city from persons")
	if err != nil {
		panic(err.Error())
	}
	defer result.Close()
	for result.Next() {
		var person Persons
		err := result.Scan(&person.Person_id, &person.Person_name, &person.Person_city)
		if err != nil {
			panic(err.Error())
		}
		//strconv.Atoi(person.Person_id) // converting A to I
		persons = append(persons, person)
	}
	json.NewEncoder(w).Encode(persons)
}

func createPerson(w http.ResponseWriter, r *http.Request) {
	fmt.Println("CHECKING")
	w.Header().Set("Content-Type", "application/json")
	stmt, err := db.Prepare("INSERT INTO persons VALUES(?,?,?)")
	fmt.Println("CHECKING 2")
	if err != nil {
		panic(err.Error())
	}
	fmt.Println("CHECKING 3")
	body, err := ioutil.ReadAll(r.Body)
	if err != nil {
		panic(err.Error())
	}
	keyVal := make(map[string]string)
	json.Unmarshal(body, &keyVal)

	fmt.Println("CHECKING 3.1")

	person_id := keyVal["person_id"]
	fmt.Println("CHECKING 3.2")
	person_name := keyVal["person_name"]
	person_city := keyVal["person_city"]

	_, err = stmt.Exec(person_id, person_name, person_city)
	if err != nil {
		panic(err.Error())
	}
	fmt.Println("CHECKING 3.3")
	fmt.Println("CHECKING 4")
	fmt.Fprintf(w, "New post was created")
}
func getPerson(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	params := mux.Vars(r)
	fmt.Println(mux.Vars(r)["pid"])
	fmt.Println("CHECKING 4")
	result, err := db.Query("SELECT person_id, person_name, person_city FROM persons WHERE person_id = ?", params["pid"])
	if err != nil {
		panic(err.Error())
	}
	fmt.Println("CHECKING 5")
	defer result.Close()
	var person Persons
	for result.Next() {
		err := result.Scan(&person.Person_id, &person.Person_name, &person.Person_city)
		if err != nil {
			panic(err.Error())
		}
	}
	json.NewEncoder(w).Encode(person)
}
func updatePerson(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	params := mux.Vars(r)
	fmt.Println(mux.Vars(r)["pid"])
	stmt, err := db.Prepare("UPDATE persons SET person_name = ?, person_city = ? WHERE person_id = ?")
	if err != nil {
		panic(err.Error())
	}
	body, err := ioutil.ReadAll(r.Body)
	if err != nil {
		panic(err.Error())
	}
	keyVal := make(map[string]string)
	json.Unmarshal(body, &keyVal)
	Person_name := keyVal["person_name"]
	Person_city := keyVal["person_city"]

	_, err = stmt.Exec(Person_name, Person_city, params["pid"])
	if err != nil {
		panic(err.Error())
	}
	fmt.Fprintf(w, "Post with ID = %s was updated", params["pid"])
}
func deletePerson(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	params := mux.Vars(r)
	stmt, err := db.Prepare("DELETE FROM persons WHERE person_id = ?")
	if err != nil {
		panic(err.Error())
	}
	_, err = stmt.Exec(params["pid"])
	if err != nil {
		panic(err.Error())
	}
	fmt.Fprintf(w, "Post with ID = %s was deleted", params["pid"])
}
