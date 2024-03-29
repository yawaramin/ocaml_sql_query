## ocaml_sql_query - functional-style SQLite queries for OCaml

Copyright 2022 Yawar Amin

This file is part of ocaml_sql_query.

ocaml_sql_query is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later
version.

ocaml_sql_query is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
ocaml_sql_query. If not, see <https://www.gnu.org/licenses/>.

### Examples

```ocaml
open Sql

(* Test DB *)
let db = Sqlite3.db_open ":memory:"
(* val db : Sqlite3.db = <abstr> *)

(* DDL query with no arguments and no return *)
let () = query db "create table people (name text not null, age int)" unit

(* Insert query with single row *)
let () = query
  db
  "insert into people (name, age) values (:name, :age)"
  ~args:Arg.[text "A"; int 1]
  unit

(* Get single column of results from DB *)
let names = query
  db
  "select name from people where rowid = :id"
  ~args:Arg.[int 1]
  @@ ret @@ text 0
(* val names : string Seq.t = <fun> *)

let () =
  query db "insert into people (name) values (:name)" ~args:Arg.[text "B"] unit

(* Get optional values *)
let ages = List.of_seq
  @@ query db "select age from people" @@ ret @@ opt int 0
(* val ages : int option list = [Some 1; None] *)

(* Map return data to a custom type on the fly *)
type person = { name : string; age : int option }

let person row = { name = text 0 row; age = opt int 1 row }
(* val person : row -> person *)

(* Assert resultset has a single row and map it *)
let person_1 = only
  @@ query db "select name, age from people where rowid = :id" ~args:Arg.[int 1]
  @@ ret person
(* val person_1 : person = {name = "A"; age = Some 1} *)

(* Assert resultset has either 0 or 1 element *)
let opt_person_1 = optional
  @@ query db "select name, age from people where rowid = :id" ~args:Arg.[int 2]
  @@ ret person
(* val opt_person_1 : person option = None *)

(* Batch insert *)

let ppl = [{ name = "B"; age = None }; { name = "C"; age = Some 3 }]

let () = batch_insert
  db
  "insert into people (name, age) values (:name, :age)"
  ppl
  (fun { name; age } -> Arg.[text name; opt int age])
  unit
```
