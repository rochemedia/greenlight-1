# Register a node
## Preparing a node identity

We start by creating a node identity, consisting of a node's seed
secret, and it's mTLS certificates we'll later use to authenticate
against Greenlight. Let's start with the seed secret: the seed secret
is a 32 byte secret that all other secrets and private keys are
derived from, as such it is paramount that this secret never leaves
your device and is only handled by the _Signer_. It is suggested to
derive the seed secret from a [BIP 39][bip39] seed phrase, so the user
can back it up on a physical piece of paper, steel plate, or whatever
creative way of storing it they can think of.

=== "Rust"
	
	Install the `bip39` and `rand` crates required for secure randomness and conversion of mnemonics. Add the following lines to your Cargo.toml
	
	```toml
	[dependencies]
	rand = "*"
	bip39 = "*"
	```

=== "Python"

	Install the `bip39` package which we'll use to encode the
	seed secret as a seed phrase:
	
	```sh
	pip install bip39
	```

Now we can securely generate some randomness, encode it as BIP 39
phrase and then convert it into a seed secret we can use:

=== "Rust"
	```rust
	use bip39::{Mnemonic, Language};

	let mut rng = rand::thread_rng();
	let m = Mnemonic::generate_in_with(&mut rng, Language::English, 24).unwrap();
	let phrase = m.word_iter().fold("".to_string(), |c, n| c + " " + n);
	
	# Prompt user to safely store the phrase
	
	seed = m.to_seed("")[0..32]  # Only need the first 32 bytes

	# Store the seed on the filesystem, or secure configuration system
	```

=== "Python"

	```python
	import bip39
	import secrets  # Make sure to use cryptographically sound randomness
	
	rand = secres.randbits(256)  # 32 bytes of randomness
	phrase = bip39.encode_bytes(rand)
	
	# Prompt user to safely store the phrase
	
	seed = bip32.phrase_to_seed(phrase)[:32]  # Only need 32 bytes
	
	# Store the seed on the filesystem, or secure configuration system
	```

=== "Javascript"
	<!-- TODO -->

Next we instantiate an mTLS identity we will use to authenticate with
Greenlight. Since we haven't registered our node with the service yet,
we will use a dummy key, that allows us to talk to the _Scheduler_ but
will not allow us to talk to any other service. No worries, once we
register the node we will get a valid certificate to authenticate.

=== "Rust"
	```rust
	use gl_client::tls::TlsConfig;

	let tls = TlsConfig::new();
	```
	
=== "Python"

	```python
	from glclient import TlsConfig
	
	# Creating a new `TlsConfig` object will automatically load the dummy identity
	tls = TlsConfig()
	```
	
=== "Javascript"
	```js
	const glclient = require('glclient');
	
	# Creating a new `TlsConfig` object will automatically load the dummy identity
	var tls = new glclient.TlsConfig();
	```

Finally, we can create the `Signer` which processes incoming signature
requests, and is used when registering a node to prove ownership of
the private key. The last thing to decide is which network we want the node to run on. You can chose between the following networks:

 - `testnet`
 - `bitcoin`

We'll pick `bitcoin`, because ... reckless 😉

=== "Rust"
	```rust
	use gl_client::signer::Signer;
	use bitcoin::Network;
	
    signer = Signer::new(secret, Network::Bitcoin, tls)
	```

=== "Python"
	<!-- TODO -->
=== "Javascript"
	<!-- TODO -->

[bip39]: https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki


## Registering a new node

Registering a node with the _Scheduler_ creates the node metadata on
the Greenlight service, including the node's identity and the public
key, and ensures everything is setup to start the node. 

In order to register a node, the client needs to prove it has access
to the corresponding private key. Since the private key is managed
exclusively by the _Signer_ we need to first instantiate that:

=== "Python"
	```python
	from glclient import TlsConfig, Signer, Scheduler
	
	# Create an mTLS identity using the default `nobody` identity, allowing us
	# to talk to the Scheduler
	tls = TlsConfig()
	
	# The seed secret used for the identity. You most likely want 
    secret = b'\x00'*32
	```

TODO:
 - Store secret on fs
 - Add code to load from fs
 - what to do with the result
   - Store cert and key on disk