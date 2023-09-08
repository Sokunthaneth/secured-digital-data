SECURED DIGITAL DATA
=
## Test Scenario
2 people are trying to have secured communication. They are User A, and User B.

> Private key is supposed to be owned, kept private, and only accessible by own user.
Public key is supposed to be owned by own user, but can be shared publicly. Having private key, we can derive public key easily, but NOT vice versa!!

When User B want to send User A some data, User B will use User A's public key, to perform data encryption. Only User A who has own private key can decrypt and read the data.

When User A want to claim data is actually sent out by himself, he can sign the data with own private key. If User B want to validate whether signed data is actully signed by User A or not, he can use User A's public key, signed data to compare with the original data.

## Pre-Requisite
- OpenSSL
- Blockchain
```
<!-- generate private key -->
openssl genrsa > private.pem

<!-- derive public key from private key -->
openssl rsa --in private.pem -pubout --out public.pem

<!-- create data.txt -->
echo "TOP SECRET! READ AND DESTROY!" > data.txt

<!-- create data-fake.txt -->
echo "TOP SECRET! BUT THIS IS FAKE!" > data-fake.txt
```


## 1. ENCRYPTION
user B can 
```
openssl pkeyutl -encrypt -pubin public.pem -in data.txt --out data.txt.enc
```

## 2. DECRYPTION
```
openssl pkeyutl -decrypt -inkey private.pem -in data.txt.enc -out decrypt-data.txt
```

## 3. SIGN
```
openssl pkeyutl -sign -inkey private.pem -in data.txt -out data.txt.signed
```


## 4. VERIFY AUTHOR
```
openssl pkeyutl -verify -inkey public.pem -pubin -sigfile data.txt.signed -in data.txt
>> Signature Verified Successfully

openssl pkeyutl -verify -inkey public.pem -pubin -sigfile data.txt.signed -in data-fake.txt
>> Signature Verification Failure
```

## 5. HASH CALCULATION (DIGEST F(x))
```
openssl dgst -sha256 data.txt.signed
SHA2-256(data.txt.signed)= d1832814383d1f44d8d9db6f8ee7754394213675fc6b257bc0e2dc714ba5cf76*
```
*\*store resulting hash into blockchain*


## PROCEDURES
Data verification in blockchain:
1. data is signed
2. signed data is calculated hash (e.g. SHA256)
3. calculated hash is store in blockchain
4. validator take the given signed data to calcuated hash again (AKA decrypt message)
5. compare validator calculated hash with blockchain stored hash (AKA compare decrypt message with original message)
   a. if match, verification succeed
   b. if not match, verification failed

<hr>

## SUMMARY
- openssl solves whether data was actually signed by User A or not
- blockchain solves whether data has been tempered or not

> Together openssl and blockchain ensure that data is actually authentic
> - created by User A
> - has not been changed by any users, **including User A!**

```
USER A
--
signed_data = fx(userA's privkey, msg)
hash = fx(signed_data, hashing_algorithm)
blockchain_hash = store(hash)


USER B
--
hash = f(given_signed_data, hashing_algorithm)
if USER B's hash == blockhain_hash:
   -> signed_data has NOT been tempered!
else:
   -> signed_data has been tempered!

verify_data_creator = fx(signed_data, userA's pubkey, data)
if verify_data_creator == TRUE:
   -> userA is the creator!
else:
   -> userA is NOT the the creator!
```