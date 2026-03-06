**# stellarfx.org**

Documentation for developers building apps using the StellarFX Rates API to retrieve fiat currency exchange rates, Stellar token prices including Afreum token prices, and cryptocurrency prices.



**Stellar FX Rates API** 

The Stellar FX Rates API provides secure access to fiat currency exchange rates and Stellar asset prices to any Stellar address via a unique on-chain Pay-As-You-Go API access model.  

The API was originally conceived as a way for 3rd party developers on the Afreum Ecosystem to easily access asset prices to use in their applications. It turns out that the API also has significant utility for anyone building apps on Stellar who needs easy access to currency exchange rates, and the exchange rates of major Stellar assets.  

The API currently provides prices for the following asset types: 

 

	**Stellar Assets** ("type":"stellar") – The last known trading price of 170 Stellar assets, including Afreum tokens such as AFR.  

		_**Supported Assets:**_ 

		https://ipfs.io/ipns/k51qzi5uqu5dlr2rb6s26f4v1eezd96fia8h51e3glyuq2gw2hiwry1am28yvv/afr_token_flexible.json 

		https://ipfs.io/ipns/k51qzi5uqu5dlr2rb6s26f4v1eezd96fia8h51e3glyuq2gw2hiwry1am28yvv/afr_token_other.json 



	**Fiat Currencies** ("type":"fiat") – The latest exchange rates of over 150 fiat currencies. 

		_**Supported Assets**_ (see currency fields): 

		https://ipfs.io/ipns/k51qzi5uqu5dlr2rb6s26f4v1eezd96fia8h51e3glyuq2gw2hiwry1am28yvv/afr_token_stable.json  



	**Cryptocurrencies** – The lates prices of popular cryptocurrencies. 

		_**Supported Assets:**_ 

		https://ipfs.io/ipns/k51qzi5uqu5dlr2rb6s26f4v1eezd96fia8h51e3glyuq2gw2hiwry1am28yvv/afr_token_external.json	 

 



**API Access** 

Send a minimum of **100,000 AFR** (Home Domain: afreum.com; Issuer: GBX6YI45VU7WNAAKA3RBFDR3I3UKNFHTJPQ5F6KOOKSGYIAM4TRQN54W) to the Stellar FX API wallet **GAL2FYLOZAVBVGUZPQU3GACIRYCZRF7CVOYUSNY5ZVZEMT53CRHLKOZQ** for 30 days of API access.  


You may send more than 100,000 AFR, in which case your API access will be pro-rated e.g. 150,000 AFR = 45 days of API access; 1,000,000 AFR = 300 days of API access. Do not send less than 100,000 AFR to the API wallet, as access is processed automatically and your funds will be lost. 



🔑 **Authentication Flow** 

- Send a minimum of 100,000 AFR to the API wallet. 

- Call /claim-access to retrieve a reusable JWT token. The token expires at the end of your access period  

- Use the JWT in the Authorization header when calling /getRates or /account-status. 

  


**📌 Endpoints** 
 

The API can be found at:  

	https://rates-api-1.stellarfx.org 
	
	https://rates-api-2.stellarfx.org 
	
	https://rates-api-3.stellarfx.org 

  


**1. /claim-access** 

Retrieve a JWT after sending at least 100,000 AFR to the API wallet. The API requires Bump Sequence transaction id from the Calling Address within 5 minutes of calling the endpoint. You usually call this endpoint once and the JWT token can be used throughout the valid 

 

_**Example Function:**_ 
	
**Note:** _Include or import StellarSDK to run this code example._ 
	 
	
	
	async function getJWT() {  
	
	  const { Horizon, Keypair, TransactionBuilder, Operation, Networks, BASE_FEE } = StellarSdk;  
	  const SERVER = 'https://rates-api-1.stellarfx.org';  
	   
	  // 0. Hardcoded secret key  
	
	  const SECRET_KEY = "[SECRET KEY]"; // Can alternatively write code to sign with Freighter, Lobstr, or build the transaction in Stellar Labs  
	  const keypair = Keypair.fromSecret(SECRET_KEY);  
	  const callerAddress = keypair.publicKey();  
	  console.log("Using hardcoded Stellar address: " + callerAddress);  
	
	
	  // 1. Load account  
	
	  console.log("Loading account from Horizon...");  
	  const server = new Horizon.Server("https://horizon.stellar.org");  
	  let account;  
	
	  try {  
	    account = await server.loadAccount(callerAddress);  
	  } catch (err) {  
	   	console.log("❌ Failed to load account from Horizon.");  
	  	console.error(err);  
	    return null;  
	  }  
	
	  // 2. Build bump-sequence transaction  
	  
	  console.log("Building bump sequence transaction...");  
	
	  const tx = new TransactionBuilder(account, {  
	    fee: BASE_FEE,  
	    networkPassphrase: Networks.PUBLIC  
	  })  
	
	  .addOperation(Operation.bumpSequence({ bumpTo: "0" }))  
	  .setTimeout(60)  
	  .build();  
	
	  // 3. Sign locally  
	  
	  console.log("Signing transaction locally...");  
	  try {  
	   	tx.sign(keypair);  
	  } catch (err) {  
	    console.log("❌ Local signing failed.");  
	    console.error(err);  
	    return null;  
	  }    
	
	
	  // 4. Submit transaction  
	  
	  console.log("Submitting transaction to Horizon...");  
	  let result;  
	
	  try {  
	   	result = await server.submitTransaction(tx);  
	  } catch (err) {  
	    console.log("❌ Transaction submission failed.");  
	    console.error(err.response?.data || err);  
	    return null;  
	  }  
	
	  const txHash = result.hash;  
	  console.log("✔ Transaction submitted. Hash: " + txHash);    
	
	  
	  // 5. Send txHash + callerAddress to backend  
	  
	  const payload = { callerAddress, txHash };  
	  console.log("Sending to /claim-access:\n" + JSON.stringify(payload, null, 2));  
	  const res = await fetch(`${SERVER}/claim-access`, {  
	    method: "POST",  
	    headers: { "Content-Type": "application/json" },  
	    body: JSON.stringify(payload)  
	  });  
	  const json = await res.json();  
	  console.log("Response from /claim-access:\n" + JSON.stringify(json, null, 2)); 
	    
	  if (!json.token) {  
	    console.log("❌ Backend did not return a JWT.");  
	    return null;  
	  }  
	  console.log("✔ JWT received.");  
	  return json.token;  
	}  
	  
	
	
	// Call the function  
	
	getJWT()  
	
	.then(jwt => {  
		console.log(jwt); // Handle the resolved value  
		// Call /getRates or /account-status endpoint with the retrieved JWT token  
	})  
	
	.catch(error => {  
		console.error(`Error fetching JWT token: `, error);  
	}); 

	 
 

_**Expected Response:**_ 
	  
	
	{ "token": "[JWT TOKEN]", "expires_in": 2329738971, "whitelisted": true, "valid_until": 4102444800000 }   
	
	 
	
_**Error Messages:**_ 

	
	You will either get a response as above or one of the following error messages: 
	
	{ error: "invalid-stellar-address" } - callerAddress should be a 56-character Stellar public key 
	
	{ error: "missing-transaction-hash" } - txHash should be a Stellar transaction hash of a recent Bump Sequence transaction 
	
	{ error: "server-misconfigured-no-jwt-secret" } - a misconfiguration on the server. Should contact Afreum by DM to @Afreum1 if you encounter this error 
	
	{ error: "transaction-not-found" } - the API was unable to validate the txHash 
	
	{ error: "transaction-source-mismatch" } - the source account of the transaction represented by the txHash should be the same as the callerAddress 
	
	{ error: "transaction-not-successful" } - the txHash is valid but the transaction was not successful 
	
	{ error: "transaction-too-old" } - the txHash is outside of the allowed 5 minute window. Typically, you should complete the Bump Sequence transaction and immediately use the txHash to retrieve your JWT token 
	
	{ error: "transaction-not-bump-sequence" } - the txHash is valid but the transaction is not a Bump Sequence transaction 
	
	{ error: "payment-required" } -  a valid AFR payment was not found 
	
	{ error: "internal-error" } - a server error occurred 

 

**2. /account-status** 

Check the status of the account and monitor how long an account still has API access. 

 
_**Example Request:**_ 
	
	
	async function getAccountStatus(publicKey,jwtToken) {  
	
	const SERVER = 'https://rates-api-1.stellarfx.org';  
	
	  const body = { callerAddress: publicKey};  
		console.log("Calling /account-status with JWT...");  
	
	  const res = await fetch(`${SERVER}/account-status`, {  
	    	method: "POST",  
	    	headers: {  
	      	"Content-Type": "application/json",  
	      	"Authorization": `Bearer ${jwtToken}`  
	    	},  
	    	body: JSON.stringify(body)  
	  });   
	
	  const json = await res.json();  
	  console.log("Response from /account-status:\n" + JSON.stringify(json, null, 2));  
	 	return json; 
	}  
	  
	
	
	// Call the function  
	
	const publicKey = '[YOUR PUBLIC KEY]'; 
	const jwtToken = '[YOUR JWT TOKEN]'; 
	
	getAccountStatus(publicKey,jwtToken)  
	.then(account_status => {  
		console.log(account_status); // Handle the resolved value  
	})  
	.catch(error => {  
		console.error('Error retrieving Account Status: ', error);  
	}); 

 

_**Expected Response:**_ 
	
	{ 
	  "callerAddress": "[CALLER ADDRESS]", 
	  "valid": true, 
	  "whitelisted": false, 
	  "expiry_iso": "2026-03-31T08:40:39.000Z", 
	  "expiry_unix": 1774946439, 
	  "remaining_seconds": 2217103, 
	  "remaining_days": 25.660915185185186, 
	  "total_days_purchased": 30 
	} 
 


_**Error Messages:**_ 
	 
	{ error: "invalid-stellar-address" } - callerAddress should be a 56-character Stellar public key 
	
	{ error: "internal-error" } - a server error occurred 
	
	{ error: "missing-or-invalid-authorization-header" } - JWT token was not sent in the authorization header 
	
	{ error: "invalid-or-expired-token" } - JWT token sent in the authorization header is invalid or it has expired. In this case, another payment must be made to the API wallet or if you have already done so, the /claim-access endpoint should be used to retrieve a fresh JWT token  

 

**3. /getRates** 

Retrieve the latest exchange rates and prices by passing callingAddress, currency, type, and JWT authorization. For type (stellar|fiat|crypto) currency can be any ISO 3 fiat currency symbol. The default is USD. The type ‘stellar’ also supports a currency of XLM. 

 
_**Example Request:**_ 
	
	 
	// getRates Function  
	
	async function getRates(jwtToken) {  
		const SERVER = 'https://rates-api-1.stellarfx.org';  
		const body = { currency: "USD", type: "fiat" };  
		console.log("Calling /getRates with JWT...");  
		const res = await fetch(`${SERVER}/getRates`, {  
	   		method: "POST",  
			  headers: {  
	      	"Content-Type": "application/json",  
	     		"Authorization": `Bearer ${jwtToken}`  
	    	},  
	    	body: JSON.stringify(body)  
	  	});  
	  	const json = await res.json();  
	  	console.log("Response from /getRates:\n" + JSON.stringify(json, null, 2));  
	}  
	  
	
	// Call getRates function using JWT 
	
	const jwtToken = '[YOUR JWT TOKEN]';   
	
	getRates(jwtToken)  
	.then(rates  => {  
		console.log(rates); // Handle the resolved value  
	})  
	.catch(error => {  
		console.error('Error retrieving Rates: ', error);  
	}); 
 


_**Expected Response:**_ 
	
	{ 
	    "data": "{\"rates\":{\"ADA\":\"2756643\",\"BCH\":\"4652721102\",\"BTC\":\"731835901482\",\"DASH\":\"349775354\",\"ETH\":\"21450737658\",\"LTC\":\"569088059\",\"USDC\":\"9999563\",\"USDT\":\"10001594\",\"WLD\":\"4186211\",\"XLM\":\"1598555\",\"XRP\":\"14414612\"},\"rate_currency\":\"USD\",\"last_update_utc\":\"2026-03-05T10:30:19Z\",\"last_update_unix\":\"1772706619\"}", 
	    "currency": "USD", 
	    "type": "crypto", 
	    "callerAddress": "GBAPFZPRAQAAQ4FLRRN2TGJHSN6DHDTD2LO755S4LBEMA7K7QSFMTW6J", 
	    "timestamp": 1772707436914, 
	    "signature": "/qcTHupibNC4rnuJl2+uLCQWFJdUuFT52w8TufSYgLTY97OhGKFnHCD5EJE5gRq8au6x2hV058Gz1dU5OxppDg==" 
	} 

 

_**Error Messages:**_ 
	
	{ error: "missing-fields" } - Required fields ‘currency’ or ‘type’ are missing 
	
	{ error: "invalid-currency-format" } - ‘currency’ should be a 3-character ISO 3 currency code such as USD, GBP, or EUR 
	
	{ error: "invalid-type" } - ‘type’ should be in lowercase and one of ‘stellar’, ‘fiat’, or ‘crypto’ 
	
	{ error: "invalid-stellar-address" } - callerAddress should be a 56-character Stellar public key 
	
	{ error: "no-data" } - no data could be delivered by the API. This could happen for multiple reasons, including malformed JSON 
	
	{ error: "internal-error" } - a server error occurred 
	
	{ error: "missing-or-invalid-authorization-header" } - JWT token was not sent in the authorization header 
	
	{ error: "invalid-or-expired-token" } - JWT token sent in the authorization header is invalid or it has expired. In this case, another payment must be made to the API wallet or if you have already done so, the /claim-access endpoint should be used to retrieve a fresh JWT token 




**Security & Data Integrity** 

The Stellar FX Rates API signs every response using an Ed25519 private key. Clients can verify these signatures to ensure the data has not been tampered with, modified in transit, or spoofed by an attacker. Verification is optional but strongly recommended for any application that relies on the integrity of rate data. 

**Why Responses Are Signed** 

Each /getRates response includes: 

	- The full payload (currency, type, timestamp, data, callerAddress) 
	- A detached Ed25519 signature over that payload 
	- A public key published on the Stellar ledger 

This allows any client to independently verify: 

	- Authenticity — the response was produced by the official oracle 
	- Integrity — the payload was not modified 
	- Freshness — the timestamp is included in the signed data 
	- Non‑repudiation — only the API can produce valid signatures 

 

_**Example Payload Verification **_

Embed or import these scripts if needed: 

	<script src="https://cdn.jsdelivr.net/npm/tweetnacl@1.0.3/nacl.min.js"></script> 
	<script src="https://cdn.jsdelivr.net/npm/tweetnacl-util@0.15.1/nacl-util.min.js"></script> 

 
	const SIGNING_ACCOUNT = "GAL2FYLOZAVBVGUZPQU3GACIRYCZRF7CVOYUSNY5ZVZEMT53CRHLKOZQ"; 
	const SIGNING_KEY_DATA_FIELD = "Afreum_Rates_Signing_Key"; 

	// Fetch Signing Public Key Function
	
	async function fetchSigningPublicKey() { 
    	const res = await fetch(`https://horizon.stellar.org/accounts/${SIGNING_ACCOUNT}`); 
	    if (!res.ok) throw new Error("Failed to fetch account"); 
	    const json = await res.json(); 
	    const encoded = json.data?.[SIGNING_KEY_DATA_FIELD]; 
	    if (!encoded) throw new Error(`Missing account data field: ${SIGNING_KEY_DATA_FIELD}`); 
	
	    // Decode raw Ed25519 public key bytes 
	    const raw = nacl.util.decodeBase64(atob(encoded)); 
	
	    // Validate length 
	    if (raw.length !== 32) { 
	      console.error("Decoded key bytes:", raw); 
	      console.error("Decoded key length:", raw.length); 
	      throw new Error("Invalid Ed25519 public key length"); 
	    } 
	    return raw; 
	} 

 
	// Verify Signed Payload Function
	
	function verifySignedRatesResponse(response, publicKey) { 
	    const { signature, ...payload } = response; 
	    const msg = JSON.stringify(payload); 
	    const msgBytes = nacl.util.decodeUTF8(msg); 
	    const sigBytes = nacl.util.decodeBase64(signature); 
	    return nacl.sign.detached.verify(msgBytes, sigBytes, publicKey); 
	} 
	
	  
	// Test the Function
	async function test() { 
	
	    // 1. Fetch the signing public key from Stellar 
	
	    const publicKey = await fetchSigningPublicKey(); 
	    console.log("Fetched signing public key:", publicKey); 
	
	  
	    // 2. Call your rates API 
	
		const jwtToken = '[YOUR JWT TOKEN]'; 
	    const res = await fetch("https://rates-api-1.stellarfx.org/getRates", { 
	      method: "POST", 
	      headers: { 
	        "Content-Type": "application/json", 
	        "Authorization": `Bearer ${jwtToken}` 
	      }, 
	      body: JSON.stringify({ currency: "USD", type: "fiat" }) 
	    }); 
	
	    const json = await res.json(); 
		console.log(json); 

	
	    // 3. Verify the signature 
	    const valid = verifySignedRatesResponse(json, publicKey); 
	    console.log("Signature valid?", valid); // If this is ‘true’ you know that the rates have not been tampered with 
	  } 
	
	  test(); 
 


⚠️ **Notes **

All requests must be made over HTTPS (443). 

JWT tokens expire at the same time your access payment; use /account-status to check validity. 

Rates must be divided by 10000000 to get floating point decimal values for UI display 

Each call to getRates, returns prices for all the supported assets of that ‘type’, giving developers instant access with a single call. Typically apps would call the endpoint and evaluate rates availability before rendering UI
