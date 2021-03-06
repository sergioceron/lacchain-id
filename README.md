# 1. Creating DID

The DID method used in LACChain ID is "ethr", this method storage the public keys
and attributes of DID into a Smart Contract which is called a DIDRegistry.
In the following url is described the structure and steps to generate a new DID.

https://github.com/uport-project/ethr-did-registry

The LACChain DID Registry has been deployed in the following addresses.

| Network                                  | Address                                                |
| -----------------------------------------| ------------------------------------------------------ |
| MainNet (id: 648529)                     |      Pending to deploy...                              |
| David19 (id: 648530)                     |      Pending to deploy...                              |
| TestNet (id: 648539)                     |      0xd6A7c915066E17ba18024c799258C8A286fFBc00        |

# 2. Adding Encryption Public Key
 
In order to send and receive Verifiable Credentials using LACChain ID, is necessary to encrypt the VC before to send it.
The public key associated for encryption of the VC must be added to the DID through the 
`` setAttribute(address identity, bytes32 name, bytes value, uint validity) `` function of DIDRegistry Smart Contract.

Example: 

```
DIDRegistry.setAttribute( 'did/pub/X25519/enc/base64', encryptionPublicKeyBase64 );
```  

The algorithm proposed to do that is the [NACL](http://nacl.cr.yp.to), which is an asymmetric key algorithm. 

# 3. Resolving a DID

In order to reconstruct and resolve a DID use the following URL:

http://34.68.56.94:8080/did/{did}

Example: 

GET  http://34.68.56.94:8080/did/did:ethr:lacchain:0x01cd0acdfb36140d0dc6a4d917693ae641821891

Response:
````
{
   "@context":"https://w3id.org/did/v1",
   "id":"did:ethr:lacchain:0x01cd0acdfb36140d0dc6a4d917693ae641821891",
   "publicKey":[
      {
         "id":"did:ethr:lacchain:0x01cd0acdfb36140d0dc6a4d917693ae641821891#owner",
         "type":"Secp256k1VerificationKey2018",
         "owner":"did:ethr:lacchain:0x01cd0acdfb36140d0dc6a4d917693ae641821891",
         "ethereumAddress":"0x01cd0acdfb36140d0dc6a4d917693ae641821891"
      },
      {
         "id":"did:ethr:lacchain:0x01cd0acdfb36140d0dc6a4d917693ae641821891#delegate-1",
         "type":"X25519KeyAgreementKey2019",
         "owner":"did:ethr:lacchain:0x01cd0acdfb36140d0dc6a4d917693ae641821891",
         "publicKeyBase64":"MDQ1NzAxYzIyYTg0ZTdiMDMyNjc2YWY3YjNlZTdiZTg2MGE5NzVhODhkNzU2NzczY2Y1Yzk2MTg4ODY0NTAyZmIwNjkwZjE1M2VjODI5YjRjOTk3NDFlOWZjMDhiNjE3MTdiMjczNWI2MTMwMTk4MGNmNjFmNTM1MmU3MzkyNGFkOA=="
      }
   ],
   "authentication":[
      {
         "type":"Secp256k1SignatureAuthentication2018",
         "publicKey":"did:ethr:lacchain:0x01cd0acdfb36140d0dc6a4d917693ae641821891#owner"
      }
   ]
}
```` 

Currently, LACChain ID can resolve the following methods and networks:

| Method                         | Network                                  |
| -------------------------------| -----------------------------------------|
| ethr                           |     MainNet                              |
| ethr                           |     David19                              |
| ethr                           |     TestNet                              |
| web                            |      *                                   |

In the future we are going to support more DID methods and their corresponding networks.
 
# 4. Authentication
 The authentication process consist in requesting a string challenge in order to sign further petitions:

### Request a challenge.
 In this step, the server will send a random string that the client must use 
 to sign with their DID and send it in headers section.
 
 URL Path: /vc/auth/{did}
 HTTP Method: GET
 
 Response:
 ```
{
    "challenge": "zW7dPhe2TiSTi25i"
}
 ```

# 5. Sending VC
 Before sending a new VC, it is necessary to encrypt with the private key generated for that purpose
 in the DID method. The encrypted VC must be encoded into an array of bytes. 
To send a new VC, is necessary to sign the keccak-256 hash challenge and attach it in the signature header.

URL Path: /vc/auth/{did}
HTTP Method: POST

Request Headers:
 ```
signature: sign(keccak256(challenge))
 ```

Request Body:
 ```
{
    "subject": "did:ethr:lacchain:0x01cd0acdfb36140d0dc6a4d917693ae641821891",
    "vc": {
        "type": "Buffer", // always Buffer
        "value": [byte]
    }

}
 ```

Response Body:
 ```
{
    result: true // or false 
    error:  in  case of result=false
}
 ```

# 6. Gathering Verifiable Credentials

 To get the list of VC associated with some DID, is necessary to sign the keccak-256 hash challenge and attach it in the signature header.
 
URL Path: /vc/{did}
HTTP Method: GET

Request Headers:
 ```
signature: sign(keccak256(challenge))
 ```

Response Body:
 ```
[
	{
		"type": "Buffer",
		"data": [encrypted buffer data bytes]
	}
]
 ```
