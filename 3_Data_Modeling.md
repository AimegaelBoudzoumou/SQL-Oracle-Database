# Data Modeling

### Content

1. Introduction
2. Overview
3. Capturing Requirements
4. Conceptual Model
5. Logical Model
6. Normalization
7. Physical Model
8. Supertypes and Subtypes

----------------------------------------------------------------------------------------------------------------------

## 1. Introduction

Data modelling is the process of taking user requirements and translating them into a table design. This is a large topic, worthy of a full course in its own right! This tutorial gives a brief overview of the process.

## 2. Overview

The basic steps for designing a database for a new application are:

- Capturing requirements
- Build the conceptual model
- Design the logical model
- Create the physical model

Building a database is an iterative process. As you go through the steps, consult with the users regularly to check it meets their needs. Often you'll need to adjust the model as you hone in on what the system must do.

## 3. Capturing Requirements

The first step in building a database is to find out what information you need to store. To complete this process, speak with people who will use the system. This could be potential customers or in-house staff in the business.

For example, say you're building a system to manage appointments for hospital clinics. This will allow patients to:

- View available appointment times & select one to attend
- View details of upcoming appointments they've booked

It must store the following information:

- The location, date, and time of the appointment
- The name of the patient
- The name of the consultant who will see the patient

These are the functional requirements. i.e. what the system must do to serve its purpose. At this stage you should also capture non-functional requirements. These define how the application works.

For example, all pages in the application must load in under two seconds. These can influence how you build your tables.

## 4. Conceptual Model

The conceptual model is a high-level overview of the information the database will store. It defines key entities in the application. Entities are real-world things the database will store details of.

For the booking system, the key entities are:

- The patient - the person seeking treatment
- The consultant - the person who will diagnose the patient's condition & prescribe treatment
- The appointment - the place and time of the consultation

## 5. Logical Model

The logical model fleshes out the details in the conceptual model. It documents which aspects of the entities the system will store. These are the attributes of the entity. You represent this in an Entity-Relationship Diagram (ERD).

At this point, we've identified the following information to store:

__Patient__
- Their name

__Consultant__

- Their name

__Appointment__
- The date & time
- The clinic name & address
- The patient and consultant attending

At this point you may identify new entities. For example, the appointment stores the clinic details. Every appointment at a clinic will be at the same place. Storing the address on each appointment duplicates these details, making data errors likely.

E.g., both the appointments below are for the PHYSIO clinic. But there's a different address for each!

| APPOINTMENT_DATETIME      | CLINIC_NAME      | CLINIC_ADDRESS
| ------------------------- | ---------------- | --------------
| 1 SEP 2018 10:00          | PHYSIO           | 1 Hospital Way
| 2 SEP 2018 12:00          | PHYSIO           | 3 Hospital Street

To avoid this, create a new entity, clinic. This stores the details of each clinic:

Clinic
- Clinic name
- Its address

The appointment will now store only the clinic's name:

Appointment
- The date & time
- The clinic name
- The patient and consultant attending

Finding dependencies like this and splitting the tables is a process called normalization. This removes redundant information, ensuring you store each fact once.

The logical model will also define the data types of each attribute. And any constraints that apply. For example, when booking an appointment, the date must be in the future.

You should also define the attributes that uniquely identify each instance of an entity. These will form the primary and unique keys for the tables.

## 6. Normalization

As described above, normalization is the process of removing redundancy in your design. So you store each fact once. This stops data errors appearing.

Normal forms are numbered, starting with first normal form. Followed by second, third, etc. up to fifth. These are usually referred to by their abbreviation, xNF where x is its number. So first normal form is 1NF and so on.

There are a few other normal forms. The most common is Boyce-Codd normal form. This is a refinement of 3NF. So it is sometimes called 3.5NF.

To be in a normal form, you must meet its requirements and those of the forms lower than it. So to reach 3NF, your tables must also be in 1NF & 2NF.

A full discussion of normal forms is outside the scope of this tutorial. For more details on these, read this [simple guide to five normal forms](https://www.bkent.net/Doc/simple5.htm)

## 7. Physical Model

Once you've built your logical model, it's time to translate this to the physical model. The output of this is the create table statements to build the database.

At this point you should take into account non-functional requirements, such as performance. This will influence which type of table you create. For example, whether to partition it or build an index-organized table.

This leads to the following tables:

```sql
create table consultants (
    consultant_id   integer,
    consultant_name varchar2(100)
);

create table patients (
    patient_id   integer,
    patient_name varchar2(100)
);

create table clinics (
    clinic_name varchar2(10),
    adress      varchar2(100)
);

create table appointments (
    appointment_id       integer,
    appointment_datetime date,
    clinic_name          varchar2(30),
    consultant_id        integer,
    patient_id           integer
);
```

## 8. Supertypes and Subtypes

At this point you may note that we've stored people's names in both the consultant and patient tables. And a person could be both a consultant and a patient! This can lead to recording different names for the same person.

To avoid this, scrap the patient and consultant tables. And create a single table to store people's details instead. For example:

```sql
create table people (
    person_id integer,
    full_name varchar2(100)
);
```

Now we have a single place to record all information about people. But there may be details specific to either consultants or patients. For example, consultants have a salary, speciality, and so on. And patients may have a hospital number, etc.

```sql
create table consultants (
    consultant_id integer,
    salary        number(10,2),
    speciality    varchar2(30)
);

create table patients (
    patient_id      integer,
    hospital_number integer
);
```

So we have a supertype/subtype relationship. A supertype is a generalization. It stores attributes common to all the subtype tables below it. A subtype is a specialization. It stores attributes specific to this instance of the parent table above it.

So person is a supertype of consultant and patient. It stores details common to everyone, such as their name and birth date. The consultant and patient tables are subtypes of people. These only store details specific to people who are a consultant or patient.

In the current requirements, there is no need for the subtype tables. So you need to review your needs to determine when you need to combine entities into a supertype. Or split them into subtypes.

Here are some guidelines for determining whether to combine or split tables:

- If two or more tables have the same columns which store the same information, you're likely missing a supertype. Consider merging them into a single table
- If one table has columns which are only apply if the row is of a certain type, consider splitting these out into subtypes

Again, identifying supertypes & subtypes is an iterative process. As you build the database, you may find you need to split a table into subtypes. Or merge two tables into one. Knowing what your application will do and store is key to choosing the correct design.
