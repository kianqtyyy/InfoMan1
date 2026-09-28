# INFOMAN1 — Week 3 Lab Answers

## Task 1 — Classify Attributes and Identify Weak Entities

### Attribute Classification

**Pet Owner**
- `owner_id` — simple attribute and primary key. It identifies each owner.
- `full_name` — composite attribute because it consists of `first_name` and `last_name`.
- `phone_number` — simple attribute.

**Pet**
- `pet_id` — simple attribute and primary key. It identifies each pet.
- `name` — simple attribute.
- `species` — simple attribute.
- `age` — simple attribute. The scenario gives age directly and does not provide a birth date from which age must be calculated, so it is not treated as a derived attribute.

**Veterinarian**
- `vet_id` — simple attribute and primary key.
- `full_name` — composite attribute because it consists of `first_name` and `last_name`.
- `specialization` — simple attribute.

**Appointment**
- `appointment_id` — simple attribute and primary key.
- `appointment_date` — simple attribute.
- `reason_for_visit` — simple attribute.

**Vaccination Record**
- `vaccine_name` — simple attribute.
- `vaccination_date` — simple attribute.

### Multivalued and Derived Attributes

No attribute is explicitly stated to be multivalued. The scenario says that a pet can have several vaccination records, but this is represented by multiple `vaccination_record` entity instances rather than by making one attribute multivalued.

No attribute is explicitly derived. In particular, `age` is given as an attribute in the scenario, and no `date_of_birth` is provided.

### Weak Entity Identification

`vaccination_record` should be modeled as a **weak entity**. A weak entity cannot be uniquely identified by its own attributes and depends on an owner entity for identification. The scenario says that each vaccination record “cannot be uniquely identified or looked up on its own” and only makes sense in relation to the specific pet it belongs to.

Therefore, `vaccination_record` depends on `pet` for its identity. Its **partial key** is `vaccine_name + vaccination_date`. The complete identifier becomes:

`pet_id + vaccine_name + vaccination_date`

This means the weak entity has an identifying relationship with `pet`.

---

## Task 2 — Specify Cardinality & Participation

### 1. Owner — Pet

Business rule: “A pet owner ... is not required to have any pets on file at a given time, but every pet must belong to exactly one owner.”

**Owner end:** `0..many`
- Cardinality: many (crow's foot)
- Participation: optional (circle)

**Pet end:** `1..1`
- Cardinality: one
- Participation: mandatory (bar)

Therefore: `Owner 0..* —— 1 Pet`

One owner may have zero or many pets, while every pet must have exactly one owner.

### 2. Pet — Appointment

Business rule: “Every appointment record ... must specify exactly one ... pet—an appointment cannot exist without both.”

**Pet end:** `0..many`
- Cardinality: many
- Participation: optional

**Appointment end:** `1..1`
- Cardinality: one
- Participation: mandatory

Therefore: `Pet 0..* —— 1 Appointment`

A pet may have no appointments or many appointments, while every appointment must be for exactly one pet.

### 3. Veterinarian — Appointment

Business rule: “A veterinarian ... can conduct multiple appointments over time or none at all.”

**Veterinarian end:** `0..many`
- Cardinality: many
- Participation: optional

**Appointment end:** `1..1`
- Cardinality: one
- Participation: mandatory

Therefore: `Veterinarian 0..* —— 1 Appointment`

A veterinarian may conduct zero or many appointments, while every appointment must have exactly one veterinarian.

### 4. Pet — Vaccination Record

Business rule: “A pet may have zero, one, or several vaccination records, and each vaccination record only makes sense in relation to the specific pet it belongs to.”

**Pet end:** `0..many`
- Cardinality: many
- Participation: optional

**Vaccination Record end:** `1..1`
- Cardinality: one
- Participation: mandatory

Therefore: `Pet 0..* —— 1 Vaccination Record`

A pet may have zero or many vaccination records, while every vaccination record must belong to exactly one pet.

---

## Task 3 — Build the Logical ERD

The logical ERD contains these entities:

- `pet_owner`
- `pet`
- `veterinarian`
- `appointment`
- `vaccination_record`

The owner and veterinarian full names are resolved into `first_name` and `last_name`.

The vaccination history is resolved as a weak entity. Its partial key is `vaccine_name + vaccination_date`. The identifying parent key `pet_id` is added to form the complete key:

`pet_id + vaccine_name + vaccination_date`

There is **no separate many-to-many relationship in the final ERD** because the scenario already models appointments as entities that each connect exactly one pet to exactly one veterinarian. Therefore, a separate junction table for Pet–Veterinarian is not required.

The exported diagram is provided separately as `erd_diagram.png`.

---

## Task 4 — Translate to Relational Schema Notation

`pet_owner(<u>owner_id</u>, first_name, last_name, phone_number)`

`pet(<u>pet_id</u>, name, species, age, owner_id*)`  
`* owner_id references pet_owner(owner_id)`

`veterinarian(<u>vet_id</u>, first_name, last_name, specialization)`

`appointment(<u>appointment_id</u>, appointment_date, reason_for_visit, pet_id*, vet_id*)`  
`* pet_id references pet(pet_id)`  
`* vet_id references veterinarian(vet_id)`

`vaccination_record(<u>pet_id</u>, <u>vaccine_name</u>, <u>vaccination_date</u>)`  
`* pet_id references pet(pet_id)`

### Keys in Vaccination Record

`pet_id + vaccine_name + vaccination_date` is the composite primary key.

`vaccine_name + vaccination_date` is the weak entity's partial key, while `pet_id` comes from the identifying parent entity.

---

## Task 5 — Key Justification & Schema Validation

### Key Justification

**1. `pet_owner.owner_id` — Surrogate/assigned identifier**

`owner_id` is used as the primary key because the scenario explicitly states that each pet owner is identified by an owner ID. Using the owner ID gives every owner a stable identifier without depending on a person's name or phone number, which could change or be shared.

**2. `appointment.appointment_id` — Surrogate/assigned identifier**

`appointment_id` is used because the scenario explicitly says that every appointment record has an appointment ID. It is a simple and direct way to uniquely identify an appointment even when several appointments have the same date, reason, pet, or veterinarian.

**3. `vaccination_record` — Composite natural/identifying key**

The vaccination record uses `pet_id + vaccine_name + vaccination_date` as its primary key rather than a standalone generated ID. This follows the weak-entity rule in the scenario: a vaccination record cannot be uniquely identified independently and needs the parent pet's key. The vaccine name and vaccination date act as the partial key.

### Schema Validation Against the Scenario

**“A veterinary clinic requires a database system to track pet owners, pets, veterinarians, appointments, and vaccinations.”**  
The schema includes `pet_owner`, `pet`, `veterinarian`, `appointment`, and `vaccination_record`.

**“A pet owner, identified by an owner ID, full name ... and phone number...”**  
`pet_owner` contains `owner_id`, `first_name`, `last_name`, and `phone_number`.

**“...full name (consisting of first name and last name)...”**  
The composite `full_name` is resolved into `first_name` and `last_name`.

**“...is not required to have any pets on file at a given time...”**  
The Owner–Pet relationship is optional on the owner side, allowing zero pets.

**“...but every pet must belong to exactly one owner.”**  
`pet.owner_id` is a foreign key to `pet_owner.owner_id`, and the relationship is mandatory for each pet.

**“A pet has a pet ID, name, species, and age.”**  
`pet` contains `pet_id`, `name`, `species`, and `age`.

**“Every appointment record tracks an appointment ID, appointment date, and reason for visit...”**  
`appointment` contains `appointment_id`, `appointment_date`, and `reason_for_visit`.

**“...and it must specify exactly one veterinarian and exactly one pet—an appointment cannot exist without both.”**  
`appointment` contains `vet_id` and `pet_id`, both referencing their parent tables, and both relationships are mandatory on the appointment side.

**“A veterinarian, identified by a vet ID, full name, and specialization...”**  
`veterinarian` contains `vet_id`, `first_name`, `last_name`, and `specialization`.

**“...can conduct multiple appointments over time or none at all.”**  
The Veterinarian–Appointment relationship is optional and one-to-many from veterinarian to appointment.

**“...the clinic tracks each pet's vaccination history, consisting of a vaccine name and vaccination date...”**  
`vaccination_record` contains `vaccine_name` and `vaccination_date`.

**“...a pet may have zero, one, or several vaccination records...”**  
The Pet–Vaccination Record relationship is optional and one-to-many from pet to vaccination record.

**“...each vaccination record only makes sense in relation to the specific pet it belongs to, meaning it cannot be uniquely identified or looked up on its own.”**  
`vaccination_record` is modeled as a weak entity identified through `pet_id`, with `vaccine_name + vaccination_date` as its partial key.

### Final Validation

All stated requirements are represented in the logical ERD and relational schema. The design uses composite attributes where required, models vaccination records as a weak entity, applies the stated optional/mandatory participation constraints, and identifies all primary and foreign keys needed to represent the case study.

---

## Self-Check

- [ ] All tasks committed with individual, meaningful commit messages
- [x] All files placed inside `week3/`
- [x] This file completed `answers.md`
- [x] Repository link pasted into Moodle (no files uploaded)