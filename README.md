# Starcity Clojure Style Guide

## Naming Conventions
<a name="naming-conventions--predicates"></a><a name="1.1"></a>
- [1.1](#naming-conventions--predicates) **Predicate Functions**: When writing a predicate function, use a `?` in the name.

> Why? A function _does_ something. In the case of predicate functions, that something is "ask a question."

```clojure
;; bad
(defn is-featured ...)
(defn featured ...)

;; good
(defn featured? ...)
```

<a name="naming-conventions--booleans"></a><a name="1.2"></a>
- [1.2](#naming-conventions--booleans) **Boolean Values**: When creating variables, Datomic schema attributes, or GraphQL schema attributes, use `is` or `has` in the name, and do not use a `?`.
> Why? We often convert data into other formats from EDN, such as JSON, which do not permit question marks. 

```clojure
;; for GraphQL attributes, use `is_*` or `has_*`
;; bad 
{:field {featured {:type Boolean}}}
{:field {featured? {:type Boolean}}}
{:field {is-featured {:type Boolean}}}

;; good
{:field {is_featured {:type Boolean}}}

;; for Datomic attributes, use `is-*` or `has-*`
;; bad
{:db/ident     :property/featured
 :db/valueType :db.type/bolean}
{:db/ident     :property/featured?
 :db/valueType :db.type/bolean}
{:db/ident     :property/is-featured?
 :db/valueType :db.type/bolean}

;;good
{:db/ident     :property/is-featured
 :db/valueType :db.type/boolean}
``` 

<a name="naming-conventions--functions"></a><a name="1.3"></a>
- [1.3](#naming-conventions--functions) **Functions**: Name functions according to the data which they produce.

```clojure
;; bad
(defn parse-account-params ...)

;; good
(defn account-params ...)
```
<a name="naming-conventions--funcs-with-side-effects"></a><a name="1.4"></a>
- [1.4](#naming-conventions--funcs-with-side-effects) **Functions with side effects**: If you must write a function that performs a side effect, use a verb describing the side effect in the name, and end the name with a `!`.

```clojure
;; bad
(create-account ...)

;; good 
(create-account! ...)

```

<a name="naming-conventions--funcs-converting-data-types"></a><a name="1.5"></a>
- [1.5](#naming-conventions--funcs-converting-data-types) **Functions which perform conversions**: When naming functions that produce their input in a different data format, use `->` in the name.

```clojure
;; bad
(defn ISOString-to-instant ...)

;; good
(defn ISOString->instant ...)
```

- [x.x](#naming-conventions--graphql-mutations) **GraphQL Mutations**: GraphQL mutations should begin with the object being mutated, followed by a verb that describes the mutation.
> Why? This allows mutations to be organized alphabetically, while still grouping mutations related to the same object together.

```clojure
;; bad
:create_member_license
{:type        :member-license
 :description "Creates a new member license"
 :args        {:params {...}}
 :resolve     :member-license/create!}

;; good
:member_license_create
{:type        :member-license
 :description "Creates a new member license"
 :args        {:params {...}}
 :resolve     :member-license/create!}
```

## Functions
<a name="functions--docstrings"></a>
- [x.x](#functions--docstrings) **Docstrings**: Use docstrings for all public functions. Great docstrings describe the expected inputs and outputs to the function, elaborating when necessary.

```clojure
;; bad - no docstring
(defn create-pending-member-license! [params]
  ...)

;; a slight improvement, but still bad.
(defn create-pending-member-license! 
  "Creates a member license with a 'pending' status."
  [params]
  ...)

;; best
(defn create-pending-member-license! 
  "Creates a member license from `params` with a 'pending' status. 
   `params` is a map of params for the new license:
    - :unit - entity id for the new license's unit
    - :rate - the rental rate for the new license
    - :term - the duration of the membership term for the new license
    - :date - the UTC corrected date on which the new license will become active"
    [params]
    ...)
```

## Code Layout and Organization

<a name="layout--two-lines-top-level"></a>
- [x.x](#layout--two-lines-top-level) **Top level forms**: Use two lines to separate all top-level forms, except for `clojure.spec.alpha` forms.

```clojure
;; bad
(def greeting-en "Hello")

(def greeting-kr "안녕하세요")

;; good
(def greeting-es "Hola")


(def greeting-sv "Hej")
```

<a name="layout--one-line-for-spec-fdef"></a>
- [x.x](#layout--one-line-for-spec-fdef) **Top level `clojure.spec.alpha/fdef` forms**: Use one line to separate a top-level function definition from its corresponding spec definition.

```clojure
;;good
(defn available-units [db community] ...)

(s/fdef available-units 
        :args (s/cat :db       td/db? 
                     :property td/entity?)
        :ret  (s/* td/entity?))
```

<a name="layout--single-line-spec-def"></a>
- [x.x](#layout--single-line-spec-def) **Single-line `clojure.spec.alpha/def` forms**: When defining several single-line specs consecutively, you may omit additional lines of whitespace.

```clojure
;; okay
(s/def :note/content string?)
(s/def :note/refs (s/+ td/entity?))
(s/def :note/children (s/* td/entity?))
(s/def :note/tags (s/* td/entity?))
```

<a name="layout--multi-line-spec-def"></a>
- [x.x](#layout--multi-line-spec-def) **Multi-line `clojure.spec.alpha/def` forms**: When defining several multi-line specs consecutively, use one line to separate each spec definition.

```clojure
;; good
(s/def ::status
  #{:active
    :inactive
    :renewal
    :canceled
    :pending})

(s/def ::role
  #{:member
    :admin 
    :onboarding 
    :applicant})
```

<a name="layout--2-space-indentation"></a>
- [x.x](#layout--2-space-indentation) **Indentation**: Use two spaces for indentation. Do not use hard tabs.

<a name="layout--align-maps"></a>
- [x.x](#layout--align-maps) **Aligning map forms**: When defining map literals, align all values in the map in relation to the map's longest key name.

```clojure
;; bad
{:username "someuser123"
 :email "user@email.com"
 :age 35}

 ;; good
 {:username "someuser123"
  :email    "user@email.com"
  :age      35}
```

<a name="layout--trailing-brackets"></a>
- [x.x](#layout--trailing-brackets) **Trailing brackets**: Place trailing brackets on one line, rather than on their own lines.

```clojure
;; bad
(defn- old-enough-to-drink?
  [person]
  (let [age (:age person)]
    (if (>= 21 age)
      true 
      false
    )
  )
)

;; good
(defn- old-enough-to-drink?
  [person]
  (let [age (:age person)]
    (if (>= 21 age)
      true
      false)))
```
