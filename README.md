## Firebase

A work in progress to contains a full cheatsheet of all the features of firebase in one and simple place
You are welcome to do PR

* [Security Rules](#security-rules)
* [Cloud Functions](#cloud-functions)
* [Firestore](#firestore)

## Security Rules
TODO

## Cloud Functions


##### Import the admin/functions and set a region of the functions
```
import * as Functions from "firebase-functions";
import * as admin from "firebase-admin";

const region = "europe-west1";
admin.initializeApp();
const functions = Functions.region(region);
```

##### Simple HTTP Trigger
```
export const onExample = functions.https.onRequest(request, response) =>
{
    return response.json({data: {success: true}});
});
```

##### Authorized User HTTP Trigger
```
function getDecodedIdToken(request, response): Promise<admin.auth.DecodedIdToken>
{
    const tokenId = request.get("Authorization").split("Bearer ")[1];

    return admin.auth().verifyIdToken(tokenId).catch((err) => response.status(401).send(err));
}

export const onUserRequest = functions.https.onRequest(request, response) =>
{
    return getDecodedIdToken(request, response).then(decoded =>
    {
        console.log("User UID:", decoded.uid);
    });
});
```


## Firestore

##### Get all the documents of a collection

```
admin.firestore()
        .collection("items")
        .get()
        .then(snapshot =>
        {
            const list = [];

            snapshot.docs.forEach(doc =>
            {
                list.push({
                    id: doc.id,
                    ...doc.data()
                });
            });

            console.log("items:", list);
        });
```