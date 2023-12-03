## Bulking up our JWKS server
The server ```main.py``` uses a pre-written table schema to query a database ```totally_not_my_privateKeys.db```.
The table schema(s):
<br>1.
```
CREATE TABLE IF NOT EXISTS keys(
    kid INTEGER PRIMARY KEY AUTOINCREMENT,
    key BLOB NOT NULL,
    exp INTEGER NOT NULL
)
```

<br>2.
```
CREATE TABLE IF NOT EXISTS users(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    email TEXT UNIQUE,
    date_registered TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP      
)
```

<br>3.
```
CREATE TABLE IF NOT EXISTS auth_logs(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    request_ip TEXT NOT NULL,
    request_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    user_id INTEGER,  
    FOREIGN KEY(user_id) REFERENCES users(id)
);
```

The ```POST:/auth``` and ```GET:/.well-known/jwks.json``` endpoints have been updated as such:
<br>

#### Under ```POST:/auth``` endpoint:
<ul>
<li>If the “expired” query parameter is not present, a valid (unexpired) key is read.</li>
<li>If the “expired” query parameter is present, an expired key is read.</li>
    <li>Logs the following details into the DB table _auth_logs_: Request IP address, Timestamp of the request, and User ID of the username.</li>
</ul>

#### Under ```GET:/.well-known/jwks.json``` endpoint:
<ul>
  <li>All valid (non-expired) private keys are read from the DB.</li>
  <li>A JWKS response is created from those private keys.</li>
</ul>

#### Under ```POST:/register``` endpoint:
<ul>
    <li>Accepts user registration details in the request body using this JSON format:{"username": "$MyCoolUsername", "email": "$MyCoolEmail"}</li>
<li>Generates a secure password for the user using UUIDv4</li>
<li>Returns the password to the user in this JSON format: {"password": "$UUIDv4"}.</li>
<li>Returned HTTP status code OK - 200</li>
<li>Hashes the password using the secure password hashing algorithm Argon2 with the configurable settings (time, memory, parallelism, key length, and salt) up to you.</li>
<li>Stores the user details and hashed password in the _users_ table.</li>
</ul>


Linting was achieved using VS Code's extension, _PyLint_.
#### Server Usage Requirements:
pip install cryptography==41.0.4
<br>
pip install pyjwt==2.8.0
<br>
pip3 install pycryptodome  // for AES encryption
