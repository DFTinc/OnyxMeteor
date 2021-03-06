#API REFERENCE

## ENROLL
###/api/v1/onyx/enroll

**Required parameters**

| parameter  | type  | description |
|---|---|---|
| api_key  | string  | This key is generated for the admin user created the first time the application is run.  Retrieve this value from api_keys collection of the app MongoDB  | 
| template  | base64encoded string  | A base64 encoded string containing the binary data of the fingerprint template generated by the Onyx SDK on a mobile device.  | 

####Return result
| key  | type  | description  |
|---|---|---|
| success  | boolean  | `true` if the provided fingerprint template was inserted into the MongoDB. `false` if a duplicate fingerprint was found.|
| userId  | string  | new userId if success was `true`, userId of the duplicate fingerprint if success was `false`  | 
| message  | string  | Enrollment result message. 'Successfully enrolled fingerprint.' or 'Duplicate fingerprint found.'  | 

The enroll endpoint will perform an identify to prevent enrolling a duplicate fingerprint.
If there is no match, the fingerprint template supplied will be inserted into the database and a new userId will be generated and returned.
If there is a match, the userId of the matching fingerprint will be returned and the supplied fingerprint template will not be saved.

## IDENTIFY
###/api/v1/onyx/identify

**Required parameters**

| parameter  | type  | description |
|---|---|---|
| api_key  | string  | This key is generated for the admin user created the first time the application is run.  Retrieve this value from api_keys collection of the app MongoDB  | 
| template  | base64encoded string  | A base64 encoded string containing the binary data of the fingerprint template generated by the Onyx SDK on a mobile device.  | 

####Return result
| key  | type  | description  |
|---|---|---|
| success  | boolean  | `true` if the provided fingerprint template matched a fingerprint in the database. `false` if no matching fingerprint was found.|
| userId  | string  | will be defined and value will be the matching userId if success was `true`, undefined if success was `false`  | 
| message  | string  | Enrollment result message. 'No match found.' or 'Found a matching userId'  | 

The identify endpoint will query the entire database for a matching fingerprint..
If there is a match, the userId of the matching fingerprint will be returned.
If there is NOT a match, `success` will be `false`


## VERIFY
###/api/v1/onyx/verify


**Required parameters**

| parameter  | type  | description |
|---|---|---|
| api_key  | string  | This key is generated for the admin user created the first time the application is run.  Retrieve this value from api_keys collection of the app MongoDB  | 
| template  | base64encoded string  | A base64 encoded string containing the binary data of the fingerprint template generated by the Onyx SDK on a mobile device.  | 
| userId  | string  | The primary key of a fingerprint template stored in the app's MongoDB. |
  
The verify endpoint will perform a one to one match of the provided fingerprint template with that of the provided userId.

####Return result
| key  | type  | description  | 
|---|---|---|
| isVerified  | boolean  | Result of the verification.    | 
| score  | number  | The numerical score of the match result.  | 

# Server Setup (Ubuntu 14.04LTS)

##Install Node.js
###node v0.10.41 REQUIRED

```
sudo npm cache clean -f
sudo npm install -g n
sudo n 0.10.41
```

##Install MongoDB https://docs.mongodb.com/v3.0/tutorial/install-mongodb-on-ubuntu/

##Install MeteorJS

```
curl https://install.meteor.com/ | sh
```

##Install Forever
```
sudo npm install forever -g
```

##Clone the project
```
git clone https://
```

##Install the Onyx node module
```
cd ~/path/to/onyxmeteor/app

npm install

cd node_modules/onyx-node/
```

##Follow the instructions in onyx-node/README.md or https://www.npmjs.com/package/onyx-node
###IDKit can be found at `~/path/to/onyxmeteor/bin/`

##Verify onyx-node setup
```
cd ~/path/to/onyxmeteor/app/node_modules/onyx-node/sample/

node onyx-verify-sample.js
```



# Build / Deploy

```
cd onyxmeteor/app

meteor build --directory ../build/

cd ../config/production/

source env.sh

cd ../../build/bundle/programs/server/

npm install

cd ../../

forever -a -l OnyxMeteor.log -o OnyxMeteorOut.log -e OnyxMeteorErrors.log start main.js

```

# Restart Server

```
cd ~/path/to/onyxmeteor/config/production/

source env.sh

cd ../../build/bundle/

forever -a -l OnyxMeteor.log -o OnyxMeteorOut.log -e OnyxMeteorErrors.log start main.js
```

# Verify deployment

```
mongo OnyxMeteor

db.api_keys.find().pretty()

```

***Sample output***

```
> db.api_keys.find().pretty()
{
	"_id" : "4BaSXiH2xHAzMy7nP",
	"owner" : "8wBx8xzBET5q5shkY",
	"key" : "51f6b93bf5f20e192195e9bc9879fd98"
}
>
```

***copy the "key" value***

Paste the api key into the enroll.js and verify.js files

```
exit

cd ~/path/to/onyxmeteor/bin/test/

npm install

vi enroll.js

vi verify.js

```

paste the api key value into the `requestJson.api_key`
make sure the **localhost:port** is correct for the request url
save the changes and exit

## Test Enroll

```
node enroll.js
```

**Sample output**

```
body: { userId: 'Y6qG46CQnfCxfJ2iJ',
  success: true,
  message: 'Successfully enrolled fingerprint.' }
```

**copy the userId**

###Error Message

Will return the userId of the existing matched fingerprint in the database.

```
body: { userId: 'QPvKAADjbQD54NEXQ',
  success: false,
  message: 'Duplicate fingerprint found.' }
```

## Test Verify

```
vi verify.js
```
Edit  verify.js and insert the userId into the requestJson.

```
node verify.js
```

***Sample output***

```
body: { isVerified: true, score: 9339 }
```

###Error message

```
onyxmeteor/bin/test$ node verify.js
body: { error:
   { error: 'not-enrolled',
     reason: 'No fingerprint enrolled.',
     message: 'No fingerprint enrolled. [not-enrolled]',
     errorType: 'Meteor.Error' },
  message: 'Error executing onyx verification.' }
```