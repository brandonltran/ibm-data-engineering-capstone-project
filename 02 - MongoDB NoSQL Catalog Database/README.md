# MongoDB NoSQL Catalog Database
> All of SoftCart's catalog data will be stored on a MongoDB NoSQL server. Create the database `catalog` and import our electronics products from `catalog.json` into a collection named `electronics`. Run test queries against the data and export the collection into a file named `electronics.csv` using only the `_id`, `type`, and `model` fields.

## 1. Install Dependencies
This assignment will be using `mongoimport` and `mongoexport` commands from the **MongoDB CLI Database Tools** library. First I will check to see if the library is installed by running one of these commands.
```console
mongoimport
```
> ```
> bash: mongoimport: command not found
> ```

Since the command is not recognized, I will need to manually install the library. First I will determine the Linux Platform and Architecture using the `lsb_release -a` command.
```console
lsb_release -a
```
> ```
> No LSB modules are available.
> Distributor ID: Ubuntu
> Description:    Ubuntu 18.04.6 LTS
> Release:        18.04
> Codename:       bionic
> ```

The processor type can be found using `uname -p`
```console
uname -p
```
> ```
> x86_64
> ```
Now I can select the appropriate package for MongoDB Database Tools from the following link: https://www.mongodb.com/try/download/database-tools

The package I will be installing is `Ubuntu 18.04 x86_64` with the `.deb` extension.

```console
sudo wget https://fastdl.mongodb.org/tools/db/mongodb-database-tools-ubuntu1804-x86_64-100.6.1.deb
```
I will use `apt install` to install the package.
```console
sudo apt install ./mongodb-database-tools-ubuntu1804-x86_64-100.6.1.deb
```

Now I will verify the installation by running the `mongoimport` command again.
```console
mongoimport
```
> ```
> 2023-02-05T15:11:01.000-0500    no collection specified
> 2023-02-05T15:11:01.000-0500    using filename '' as collection
> 2023-02-05T15:11:01.000-0500    error validating settings: invalid collection name: collection name cannot be an empty string
> ```
The command is now recognized, meaning the package has successfully been installed.

## 2. Import Catalog Data From JSON
```console
sudo service mongodb start
```

I will start by creating a database `catalog` and a collection named `electronics` using the MongoDB CLI.
```console
use catalog
db.createCollection("electronics")
```

Now I can download the catalog data and load it into the collection.
```console
sudo wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DB0321EN-SkillsNetwork/nosql/catalog.json
mongoimport --host localhost --port 27017 --db catalog --collection electronics --authenticationDatabase admin --username root --password PASSWORD catalog.json
```
> ```
> 2023-02-05T16:31:01.726-0500    connected to: mongodb://localhost/
> 2023-02-05T16:31:01.751-0500    438 document(s) imported successfully. 0 document(s) failed to import.
> ```

## 3. Run Test Queries
To display a list of all databases, I can run the following command using the MongoDB CLI:
```console
db.adminCommand( { listDatabases: 1 } )
```
> ```
> {
>         "databases" : [
>                 {
>                         "name" : "admin",
>                         "sizeOnDisk" : 32768,
>                         "empty" : false
>                 },
>                 {
>                         "name" : "catalog",
>                         "sizeOnDisk" : 40960,
>                         "empty" : false
>                 },
>                 {
>                         "name" : "config",
>                         "sizeOnDisk" : 69632,
>                         "empty" : false
>                 },
>                 {
>                         "name" : "local",
>                         "sizeOnDisk" : 32768,
>                         "empty" : false
>                 }
>         ],
>         "totalSize" : 176128,
>         "ok" : 1
> }
> ```

Now I will display a list of all collections in our `catalog` database.
```console
use catalog
db.runCommand( { listCollections: 1 } )
```
> ```
> {
>         "cursor" : {
>                 "id" : NumberLong(0),
>                 "ns" : "catalog.$cmd.listCollections",
>                 "firstBatch" : [
>                         {
>                                 "name" : "electronics",
>                                 "type" : "collection",
>                                 "options" : {
> 
>                                 },
>                                 "info" : {
>                                         "readOnly" : false,
>                                         "uuid" : UUID("ed60269b-0bd7-41f1-87e2-d7c405b446fe")
>                                 },
>                                 "idIndex" : {
>                                         "v" : 2,
>                                         "key" : {
>                                                 "_id" : 1
>                                         },
>                                         "name" : "_id_",
>                                         "ns" : "catalog.electronics"
>                                 }
>                         }
>                 ]
>         },
>         "ok" : 1
> }
> ```

## 4. Create Index
I will use the following command to create an index for our collection:
```console
db.electronics.createIndex({"type" : 1})
```
> ```
> {
>         "createdCollectionAutomatically" : false,
>         "numIndexesBefore" : 1,
>         "numIndexesAfter" : 2,
>         "ok" : 1
> }
> ```

Now that our collection is indexed, I will run a few more test queries. I'll start by displaying the record count for product type `laptops`
```console
db.electronics.find( {"type":"laptop"} ).count()
```
> ```
> 389
> ```

Next I will display the record count for product type `smart phones` with screen size of `6 inches`
```console
db.electronics.find( {"type":"smart phone", "screen size":6} ).count()
```
> ```
> 8
> ```

Now I will write an aggregration query to display the average `screen size` of product type `smart phones`
```console
db.electronics.aggregate([{$match: {"type": "smart phone"}},{$group: {_id:"$type", avg_val:{$avg:"$screen size"}}}])
```
> ```
> { "_id" : "smart phone", "avg_val" : 6 }
> ```

## 5. Export Data
Finally, I will export the `electronics` collection into a file named `electronics.csv` using on the `_id`, `type`, and `model` columns.
```console
mongoexport --host localhost --port 27017 --authenticationDatabase admin --username root --password PASSWORD --db catalog --collection electronics --fields _id,type,model --out electronics.csv
```
> ```
> 2023-02-05T18:22:54.806-0500    connected to: mongodb://localhost/
> 2023-02-05T18:22:54.819-0500    exported 438 records
> ```

## About This Lab
##### Environment/IDE
This portion of the project will be using the Cloud IDE based on Theia and MongoDB running in a Docker container.
##### Tools/Software
- MongoDB Server
- MongoDB Command Line Backup Tools

[<kbd> <br> ← Previous Assignment <br> </kbd>](/01%20-%20MySQL%20Online%20Transactional%20Processing%20Database)
[<kbd> <br> → Next Assignment <br> </kbd>](/03%20-%20PostgreSQL%20Staging%20Data%20Warehouse)
