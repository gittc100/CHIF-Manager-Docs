# CHIF Manager

This is the working document for the Entire CHIF Manager Project

## Working Documents

[Working Design Document](https://www.figma.com/file/agzWzQDvNUJvUcSf0EPDvY/C-Hear-Auth-Wires?node-id=0%3A1)

[Other Documentation](https://github.com/C-Hear/documentation)

## Client Screen Shots

### Home Page

![Home Page](./home.png)

### File Builder Page

![Builder Page](./builder.png)

### Library Page

![Library Page](./library.png)

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

1. Node.js (https://nodejs.org)
2. NPM (https://www.npmjs.com/)

### Installing

#### Layout

`/client` is the Static Client in React <br>
`/server` is the API in Express

#### Running the app locally

clone the repo to your machine and run the following commands:

##### Install Dependencies

1. `npm install` from `/client`
2. `npm install` from `/server`

##### .env files

1. Reach out to a team member for the .env files for the client and the server, place them in their respective dir's `/client` and `/server`
2. Obtain the ssl certs for gcp sql connection and place them in `/server/api/database/testCerts`

##### Run Client

1. `npm start` from `/client`

##### Run Server

1. Depending on the `/server/.env` file configuration you can connect to a local database or the gcp test database.

2. if you wish to connect to a local database you must download and install pgAmdin https://www.pgadmin.org/
3. Once you set up your pgadmin and root user, create a postgres user name `chear`. This will be your local database user for this project.
4. Modify the .env file variables: `NODE_ENV=localhost TEST_TYPE=local`, NODE_ENV controls the local connection for the server and TEST_TYPE controls the connection for the automated testing.
5. Create the local database by running `npm run initialize-database` from `/server`
6. Lastly run `npm run dev` from `/server` and your local server will be running on port 3001.
7. To start the automated testing file run `npm run test` from `/server`.
8. If you wish to run the server/test using the google test sql instance modify the .env file variables: `NODE_ENV=developement TEST_TYPE=gcp_test`.

##### Ports

1. Client can be accessed from http://localhost:3000/ or http://localhost:3001/

2. Server can be accessed from http://localhost:3001/api/

The client is served from the api as a static file during `production` on port `3001`, use port `3000` for `local developement`.

## CLIENT

## API Documentation

### 3rd PARTY API's

#### [Auth0](https://manage.auth0.com)

Provides Authorization/Authentication Service

Production Account Email: appuser@c-hear.com

#### [MailGun](https://app.mailgun.com/app/dashboard)

Provides Mailer Service

Production Account Email: appuser@c-hear.com

### Data Model (Postgres SQL)

#### users

---

```
{
  id uuid,
  user_name text,
  phone text,
  name text,
  gender char, (m/f/o)
  email text,
  dob date,
  create_date date,
  update_date date
}
```

#### organizations

---

```
{}
	id uuid DEFAULT gen_random_uuid() primary key,
	organization_name text not null,
	organization_code text not null,
	payment_account_id text,
	payment_subscription_id text,
	chif_directory text,
	address_1 text,
	city_1 text,
	state_1 text,
	zip_1 text,
	country_1 text,
	address_2 text,
	city_2 text,
	state_2 text,
	zip_2 text,
	country_2 text,
	create_date date,
	update_date date
}
```

#### file_structure

---

Directories and file asscoiation with the organization

```
{
	id uuid DEFAULT gen_random_uuid() primary key,
	 org_id text not null, -- (files/dirs)
    file_uuid uuid, -- (files)
    item_name text, -- (files/dirs) root directory name = 'root'
    mediaLink text, -- (files)
    file_size text, -- (files)
    timeCreated text, -- (files/dirs)
    parent_id uuid, -- parent directory id (files/dirs) "root" = NULL
    item_type text -- (directory/file)
}
```

#### uuids

---

File uuids associated with the organization

```
{
	id uuid DEFAULT gen_random_uuid() primary key,
	org_id text not null,
	uuid_value uuid
}
```

#### user_org_rel

---

Relationship between users and organizations

```
{
  id uuid,
  user_id uuid,
  org_id uuid,
  user_type text, ("admin"/"user") (admin/user)
  is_owner bit, (0/1) (not owner/owner)
  is_approved bit, (0/1) (waiting/approved)
  create_date date,
  update_date date
}
```

#### auth_user_token

---

Internal Authorization Session Token

```
{
  token text,
  email text,
  create_date timestamp
}
```

#### external_session_token

---

External API Authorization Session Token

```
{
  token text,
  token_name text,
  org_id uuid,
  user_id uuid,
  create_date timestamp,
  expiration_date timestamp + '1 year'
}
```

#### queue_tasks

---

internal queue task overlaying pg-boss queue

```
{
  id uuid,
  org_id uuid,
  job_id uuid,
  job_name text,
  queue_status text,
  job_data jsonb,
  response jsonb,
  retry_limit int,
  retry_count int,
  create_date timestamp
}
```

### SCHEMA FILE Postgres (SQL)

```
drop schema if exists chear;
create schema chear;
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
set search_path=chear;
DROP TABLE if exists chear.organizations cascade;
CREATE TABLE chear.organizations
(
	id uuid DEFAULT gen_random_uuid() primary key,
	organization_name text not null,
	organization_code text not null,
	payment_account_id text,
	payment_subscription_id text,
	chif_directory text,
	address_1 text,
	city_1 text,
	state_1 text,
	zip_1 text,
	country_1 text,
	address_2 text,
	city_2 text,
	state_2 text,
	zip_2 text,
	country_2 text,
	create_date date,
	update_date date
);
create unique index "UniqueOrgName" on chear.organizations(organization_name);
create unique index "UniqueOrgCode" on chear.organizations(organization_code);
DROP TABLE if exists chear.file_structure cascade;
CREATE TABLE chear.file_structure
(
	id uuid DEFAULT gen_random_uuid() primary key,
	org_id text not null,
	file_uuid uuid,
	item_name text,
	mediaLink text,
	file_size text,
	timeCreated text,
	parent_id uuid,
	item_type text
);
DROP TABLE if exists chear.uuids cascade;
CREATE TABLE chear.uuids
(
	id uuid DEFAULT gen_random_uuid() primary key,
	org_id text not null,
	uuid_value uuid
);
DROP TABLE if exists chear.users cascade;
CREATE TABLE chear.users
(
	id uuid DEFAULT gen_random_uuid() primary key,
	user_name text,
	phone text,
	name text,
	gender char(1),
	email text not null,
	dob date,
	create_date date,
	update_date date
);
create unique index "UniqueEmail" on chear.users(email);
create unique index "UniqueUserName" on chear.users(user_name);
DROP TABLE if exists chear.user_org_rel cascade;
CREATE TABLE chear.user_org_rel
(
	id uuid DEFAULT gen_random_uuid() primary key,
	user_id uuid REFERENCES chear.users(id),
	org_id uuid REFERENCES chear.organizations(id),
	user_type text,
	is_owner bit(1),
	is_approved bit(1),
	create_date date,
	update_date date
);
create unique index user_org_rel_idx1 on chear.user_org_rel(org_id,user_id);
CREATE TABLE chear.external_session_token
(
	token text primary key,
	token_name text,
	org_id uuid,
	user_id uuid,
	create_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
	expiration_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP + '1 year'
);
CREATE TABLE chear.auth_user_token
(
	token text primary key,
	email text,
	create_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE chear.queue_tasks
(
	id uuid DEFAULT gen_random_uuid() primary key,
	org_id uuid,
	job_id uuid,
	job_name text,
	queue_status text,
	job_data jsonb,
	response jsonb,
	retry_limit int,
	retry_count int,
	create_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);
```

### ENDPOINTS

MANAGER API Testing ENDPOINTS (all):

External API Production ENDPOINTS (prod):

[![External API Production ENDPOINTS - Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/05ed2698fb9f65e502d4)

Prod Domain - [https://manager.c-hear.com](https://manager.c-hear.com)

### EXTERNAL API

[https://github.com/C-Hear/chif-tutorials/blob/master/README_EXTERNAL_API.md](https://github.com/C-Hear/chif-tutorials/blob/master/README_EXTERNAL_API.md)

### INTERNAL API

#### SERVER CLIENT ENDPOINT

| Method | Endpoint | Description          | Body |
| ------ | -------- | -------------------- | ---- |
| GET    | `/*`     | Serve Static Client. | null |

#### USER ENDPOINTS

| Method | Endpoint                       | Description                           | Body                                                                                                  |
| ------ | ------------------------------ | ------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| GET    | `/api/users/user_id/:id`       | Returns a single user via user id.    | null                                                                                                  |
| GET    | `/api/users/user_email/:email` | Returns a single user via user email. | null                                                                                                  |
| PUT    | `/api/users/:id`               | Updates a single user via user id.    | `{ user_name: TEXT, phone: TEXT, name: TEXT, gender: CHARACTER, dob: DATE, organization_code: TEXT }` |
| POST   | `/api/users`                   | Post a new user.                      | `{ email: TEXT }`                                                                                     |

#### USER/ORG Relation ENDPOINTS

| Method | Endpoint                                            | Description                                   | Body                                                                                |
| ------ | --------------------------------------------------- | --------------------------------------------- | ----------------------------------------------------------------------------------- |
| GET    | `/api/user-org-rel/user_id/:user_id`                | Returns all Relations via user id.            | null                                                                                |
| GET    | `/api/user-org-rel/user_id/:user_id/org_id/:org_id` | Returns all Relations via org id and user id. | null                                                                                |
| GET    | `/api/user-org-rel/list/org_id/:org_id`             | Returns all Relations via org id.             | null                                                                                |
| POST   | `/api/user-org-rel`                                 | Post a new Relation.                          | `{ user_id bigint, org_id bigint , user_type text, is_owner bit, is_approved bit }` |
| PUT    | `/api/user-org-rel/user_id/:user_id/org_id/:org_id` | Updates a single user via user id and org id. | `{ user_type text, is_owner bit, is_approved bit }`                                 |
| DELETE | `/api/user-org-rel/user_id/:user_id/org_id/:org_id` | Delete a single user via user id and org id.  | null                                                                                |

#### ORGANIZATION ENDPOINTS

| Method | Endpoint                                                         | Description                                  | Body                                                                                                                                                                                                                                                                  |
| ------ | ---------------------------------------------------------------- | -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GET    | `/api/organizations/org_id/:org_id`                              | Returns Org via org id.                      | null                                                                                                                                                                                                                                                                  |
| GET    | `/api/organizations/drop/user_id/:user_id`                       | Returns all Orgs via user id.                | null                                                                                                                                                                                                                                                                  |
| POST   | `/api/organizations`                                             | Post a new Org.                              | `{ organization_name text, organization_code text, chif_directory text, file_structure text, address_1 text, city_1 text, state_1 text, zip_1 text, country_1 text, address_2 text, city_2 text, state_2 text, zip_2 text, country_2 text }`                          |
| PUT    | `/api/organizations/:id/user/:user_id`                           | Updates a single org via user id and org id. | `{ organization_name text, organization_code text, chif_directory text, file_structure text, address_1 text, city_1 text, state_1 text, zip_1 text, country_1 text, address_2 text, city_2 text, state_2 text, zip_2 text, country_2 text, payment_account_id text }` |
| PUT    | `/api/organizations/file_structure/org_id/:org_id/user/:user_id` | Update File Structure.                       | `{ varies }`                                                                                                                                                                                                                                                          |

#### MAIL ENDPOINTS

| Method | Endpoint    | Description       | Body                                                                                        |
| ------ | ----------- | ----------------- | ------------------------------------------------------------------------------------------- |
| POST   | `/api/mail` | Send mail Invite. | `{ sender: email, receiver: email, org_name: string, org_code: string, user_name: string }` |

#### FILE ENDPOINTS

| Method | Endpoint                                           | Description                                                                                                            | Body                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------ | -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| POST   | `/api/files/upload_chif`                           | Upload Existing File.                                                                                                  | `form-data: {CHIF_FILE: file.chif } params: { org_dir, file_path, user_id, org_id }`                                                                                                                                                                                                                                                                                                                                           |
| POST   | `/api/files/temp_file/build/:org_dir`              | build and send location temp file for preview.                                                                         | `form-data: { imageFileName: file.jpeg, audioFileName: file.mp3, textFileName: file.txt, }`                                                                                                                                                                                                                                                                                                                                    |
| POST   | `/api/files/perm_file/build/:org_dir`              | build permanent file and store in gcp.                                                                                 | form-data: `{ imageFileName: 'Local Image File', audioFileName: 'Local Audio File', textFileName: 'Local Text File', uploadedCHIFFileNAME: 'File Name',tags: 'Stringified JSON Array',author: 'string',valid_domains: 'Stringified JSON Array',alt: 'string',user: 'Stringified JSON Object (User Defined)',valid_range_start: 'Date Formate (YYYY-MM-DDTHH:MM:SSZ)',valid_range_end: 'Date Formate (YYYY-MM-DDTHH:MM:SSZ)' }` |
| GET    | `/api/files/GET_CHIF/dir/:org_dir/file/:file_name` | Returns a single file associated with organization directory in gcp.                                                   | null                                                                                                                                                                                                                                                                                                                                                                                                                           |
| POST   | `/api/files/get_files/:ids`                        | Check Job Queue for job Completion/Failure (ids is a comma seperated list of ids to check will receive a list of jobs) | null                                                                                                                                                                                                                                                                                                                                                                                                                           |
| POST   | `/api/files/ORG_DIR`                               | post new org buckets to Google Cloud Storage                                                                           | `{ org_dir: text }`                                                                                                                                                                                                                                                                                                                                                                                                            |
| GET    | `/api/files/GET_CHIF/dir/:org_dir/file/:file_name` | Returns a single file associated with organization directory in gcp.                                                   | null                                                                                                                                                                                                                                                                                                                                                                                                                           |
| GET    | `/api/files/CHIF/metadata/:org_dir/:file_name`     | Returns all file metadata.                                                                                             | null                                                                                                                                                                                                                                                                                                                                                                                                                           |
| DELETE | `/api/files/delete_chif`                           | Delete a single permanent file in gcp bucket.                                                                          | query params: `{ org_dir, file_name, file_path, org_id, user_id }`                                                                                                                                                                                                                                                                                                                                                             |
| GET    | `/api/files/file_events/:uuid/:name:/:org_id`      | Returns events associated with file.                                                                                   | null                                                                                                                                                                                                                                                                                                                                                                                                                           |
| GET    | `/api/files/exception_file/:uuid/:org_id`          | Returns exceptions for CHIF File.                                                                                      | null                                                                                                                                                                                                                                                                                                                                                                                                                           |
| POST   | `/api/files/block_file/:uuid/:org_id`              | Blocks Chif File.                                                                                                      | `{ code: text, reason: text, }`                                                                                                                                                                                                                                                                                                                                                                                                |
| DELETE | `/api/files/unblock_file/:uuid/:org_id`            | Un Blocks Chif File.                                                                                                   | null                                                                                                                                                                                                                                                                                                                                                                                                                           |
| GET    | `/api/files/read_tmp`                              | Returns tmp directory contents and total content size                                                                  | null                                                                                                                                                                                                                                                                                                                                                                                                                           |
| DELETE | `/api/files/delete_tmp`                            | Deletes tmp directory contents and returns tmp directory contents and total content size                               | null                                                                                                                                                                                                                                                                                                                                                                                                                           |

#### PAYMENT ENDPOINTS

| Method | Endpoint                                                              | Description                                                       | Body                                                              |
| ------ | --------------------------------------------------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------- |
| GET    | `/api/payment-processor/get_products`                                 | Returns all associated products. (more for internal use)          | null                                                              |
| GET    | `/api/payment-processor/get_all/:id`                                  | Returns account information per account id for multiple accounts. | null                                                              |
| GET    | `/api/payment-processor/subscription_id/:id`                          | Returns single subscription by subscription id.                   | null                                                              |
| GET    | `/api/payment-processor/account_id/:act_id/uncancel_subscription/:id` | Re Activate subscription prior to deactivation.                   | null                                                              |
| POST   | `/api/payment-processor/update_subscription`                          | Update Subscription.                                              | `{ subscription_id: text, product_path: text, account_id: text }` |
| POST   | `/api/payment-processor/create_subscription`                          | Create Subscription.                                              | `{ account_id: text, product_path: text, org_id: text }`          |
| DELETE | `/api/payment-processor/account_id/:act_id/subscription_id/:id`       | Cancel Subscription.                                              | null                                                              |

## Built With

-   [React](https://reactjs.org/) - Client Framework
-   [Express](https://expressjs.com/) - Web API Framework
-   [Node-Postgres](https://node-postgres.com/) - SQL Database
-   [Auth0](https://auth0.com/) - Authentication Process
-   [Mailgun](https://www.mailgun.com/) - Mailing System
-   [GCP](https://cloud.google.com/) - Cloud App Engine Deployment and File/Database Storage
