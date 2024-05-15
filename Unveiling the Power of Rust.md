<style>
:root {--r-code-font: "FiraCode Nerd Font";}
.reveal .hljs {min-height: 50%;}

.reveal code.rust.small {
  font-size: 0.75em;
  line-height: 1.5em;
}

%%  .reveal code.rust {
  font-size: 1.5em;
  line-height: 1.5em;
}
.reveal code.python {
  font-size: 1.5em;
  line-height: 1.5em;
}

.reveal code.javascript {
  font-size: 1.5em;
  line-height: 1.5em;
} 
.reveal code.error {
	color="#9900FF";
	  font-size: 1.5em;
	  line-height: 1.5em;
} %%

</style>

## Unveiling the Power of Rust
![[rust-logo.png]]
notes:
Welcome my name is Ryan, I am based in Pretoria and the tech stack that I work on at BBD is python and C# based. But If you had to look at my personal GitHub all you would see it overflowing with Rust code. The main reason for this is because over the last 2-3 years I have been exposed to benefits of rust in the real world as well as a lot of the more theoretical benefits from the conferences that I have attended. So today I will be sharing my passion of rust with you and hopefully convincing you as a developer or even as a team to consider Rust for your next project

---
# Rust

Rust is a statically typed, low level, multi-paradigm programming language.

notes:
Now I mean I could just rattle of this dictionary definition but I am sure that we can all google that and I would have already lost half of you guys by the statically typed. 
So what am I actual going to speak about?

1 Errors
2 where we can use it 
3 Cost related 

This was the original:
Rust is a statically typed, low level, multi-paradigm systems programming language renowned for its performance, memory safety, fearless concurrency, zero-cost abstractions, pattern matching, compiler-enforced ownership model, macro expansion, lifetimes, and trait system.


---

# Safety

notes: 
So I am not going to go into memory safety and how Rust achieves this without a garbage collector but rather how the compiler forces you to make better decisions and prevents you from poor error handling.

---

I'm TIRED of errors like this: 
- `json decoding error on line 1`
- `"unexpected ; in query"`
- `NullPointerException` (or `NoneType has no atribute`)
- `TypeError: "x" is not a function`

notes:

Now I am sure that we have all been in this situation:
Trawling through thousands and thousands of log lines just to find one of these abysmal errors.   
Lets have a look at a simple example showing some code 


---
## Python Example

```Python 
account = get_account_detail_from_server("username")  # Make request to api etc.  
card = account["card1"]  
balance = card["bal"]  
bal_value = float(balance)
```



notes: 

Now I am sure that as developers we are all familiar with this situation. We make an API call, we get some data from it and we extract the data we want and then we can do some processing on it. 

Now this code works perfectly right?
OOF it is 4:00 AM 
Production is down 
15 missed calls from the boss 
Looks like I am going to need brush up my CV 

---

## Well lets rewind a bit and dissect the issues this code.


---
#### Unsafe Assumptions 

```Python[1|2|3|4]
account = get_account_detail_from_server("username")  # Make request to api etc.  
card = account["card1"]  
balance = card["bal"]  
bal_value = float(balance)
```

notes: 
The main issue is unsafe assumptions
Further more these will only be caught at runtime.
1. assume that the get_account_detail_from_server is successful
2. the structure of the object returned is the same as the last time we used it
3. meaning it has card1
4. card1 has a bal 
5. and that bal can be converted to a float
So lets look at a rust conversion of this 

---
### Bad Rust Conversion
```rust
f64::from_str(
    get_account_detail_from_server("username").unwrap()
	    .get("card1").unwrap()
        .get("bal").unwrap()
).unwrap()
```
Note the usage of `unwrap()`

notes:

This rust version it is just as unsafe. 
Well at least the 4 places where it can crash are explicitly stated. 
The only thing that we can see from this is that rust makes it explicit when we are introducing a crash.
Now if we want our app to never crash we need to use alternatives for the unwrap's. So what are these alternatives.

---
#### A verbose rust solution

```rust small[]
if let Some(account) = get_account_detail_from_server("username") {  
    match account.get("card1") {  
        Some(card) => {  
            match card.get("bal") {  
                Some(balance) => {  
                    match f64::from_str(balance) {  
                        Ok(balance) => balance,  
                        Err(_) => { /*Unable to parse balance*/ }  
                    }  
                },  
                None => { /*bal value not found in object*/ }  
            }  
        },
        None => { /*card1 is not found in object*/ }  
    }  
} else { /*Unable to get account details from server*/ }
```

notes: 

Here is a verbose version.
Now this is quite frankly difficult to read. 
Yet one thing is made abundantly clear Rust will not allow you write unsafe code from the start.
You must handle all errors.

---
#### More idiomatic rust code

```rust
let account_response = get_account_detail_from_server("username")?;  
let card = account_response.get("card1").ok()?;  
let bal = card.get("bal").ok()?;  
let bal_value = f64::from_str(bal)?;
```

notes: 

Now this is what I am talking about this idiomatic solution keeps us on the 'happy path' and allows the error to be propagated upwards. this is done via the  the `?` (try operator) and will causing the function to return an error immediately if it occurs .


---
## So Where Can We Use Rust
Everywhere Of Course

notes:

From bare-metal chips, through backend code, AWS lambdas, and frontend and mobile and app development

---

|                |                                                            |
|----------------|------------------------------------------------------------|
| Web Frontend   | Sycamore(Wasm)                                             |
| Web Backend    | Axum                                                       |
| Games          | Bevy                                                       |
| Mobile/Desktop | Tauri                                                      |
| Sql            | SqlX(PostgreSQL, MySQL, MariaDB, SQLite)                   |
| Graphics       | Wgpu(Vulkan, Metal, DirectX 12, OpenGL ES, WebGPU, WebGL2) |

notes:

---

|                 |                             |
|-----------------|-----------------------------|
| AWS, Azure, GCP | First class support         |
| Linux           | Used In Kernel              |
| Windows         | Security critical processes |

notes:

First-class support on AWS, Azure, GCP
Rust became the third officially supported language the others being assembly and C, No CPP support
Windows has even started rewriting some of their Security critical processes in rust, guess the rewrite in rust meme is there for a reason.

---


# Why Use Rust

notes:

Now that we have seen a bit about error handling and where we can use it lets look a bit at the developer experience. 

---
## Similar syntax 

```rust
println!("Hello world");
```

notes:

Rust has the familiar c-like syntax that will be super easy to pick up as java, go, javascript or even python developer.

---
## Expressive Thinking

```rust
// statement:
let mut output; 
if option { 
    output = thing1;
} else {
    output = thing2;
}
// expression:
output = if option { thing1 } else { thing2 };
```

notes: 

Semicolons finally have proper MEANING, so when we do not include the ; it is the same as stating return x; .
Helps to reduce the usage of mutability

---
## No trade off between ergonomics and speed 

```rust
let product_of_cubes: u32 = (1..19)
    .map(|x| x.pow(3))
    .product();
```

notes:

In Rust  we can have our cake and eat it as there is no need to chose between high level features and low level speed. 

Here I have a functional-style iterator but this is just one of many zero-cost abstractions. Cool what exactly is a zero-cost abstraction you may ask. Well in essence it means that you code will get compiled down to simple loops that are easy for the CPU to understand.

---
## Nulls no more

```rust
let possibly_a_number = Some(1);

possibly_a_number
    .map(|n| n + 1)
    .unwrap_or(0);
```

notes:

Optional's replace nulls in rust.
All iterator functions can be used on optional's. 

---
### Enums Make Invalid States Impossible 

```rust
enum Condition { Healthy, Recovery, Sick }
enum Breed { Labrador, Poodle, GermanShepherd, Bulldog, Husky }
struct Dog { name: String, condition: Condition, breed: Breed }
```
```rust
let dog = Dog {
    name: "Buddy",
    condition: Condition::Sick,
    breed: Breed::Husky
}
```

---

## Complex Macro System

``` html[]
let page = html! {
  <html>
  <head>
	  <title>"My blog"</title>
  </head>
  <body>
	  <div id="my_div"></div>
  </body>
  </html>
};
```


<https://crates.io/crates/html-to-string-macro>

notes:

Would you like to write html inside your rust code? Of course you would, and you get syntax highlighting and compile checking for FREE.

Imagine being able to bring the power of rust to other programming languages. Wait we can introducing compile time macros. These provide rust with yet another super power. In this example we are able to compile time check our html to see if there are errors in it., check out YEW if you want to get more info on this.

---

### Compile-time Evaluation

```rust[]
let account = sqlx::query!("SELECT name, id FROM account")
    .fetch_one(&mut pool)
    .await?;
```
<https://crates.io/crates/sqlx>

notes:

We can use the macro, `sqlx::query!` to achieve compile-time syntactic and semantic verification of the SQL, with an output to an anonymous record type where each SQL column is a Rust field (using raw identifiers where needed).


---
### Performance, Portability, Profit 

So we all know time is money. 

notes: 

But speed should not be the only metric we look at.  

---

![[stats.jpg]]

notes:
So in this table we have a comparison between various programming languages, the relative time that it run a program as well as the amount of energy that.
So this is a bit complex lets look at at a bit more condensed version 

https://greenlab.di.uminho.pt/wp-content/uploads/2017/10/sleFinal.pdf

---

| Language | Time |
| -------- | ---- |
| C        | 1x   |
| Rust     | 1.1x |
| C++      | 1.4x |
| Java     | 2x   |
| C#       | 3x   |
| Js       | 6x   |
| Python   | 70x  |

notes: 

So all performance tests should be taken with a grain of salt.
But there is a vast difference between Python and Rust.

---

### Rust has many architectures as build targets

notes:
Why does this matter, mitigates the ideology of write once and run anywhere. 

---

## ARM Lambdas Cost 

| x64 price/ms   | ARM price/ms   |
|----------------|----------------|
| \$0.0000000021 | \$0.0000000017 |

https://aws.amazon.com/lambda/pricing/

(eu-west-2, price for 128MB)

---
### Summary

notes:
In summary we have can see that Rust is not only faster, allows ergonomic code, proper errors, cheaper and more environmentally friendly. Ok Hopefully I have sold you so: 

---

###  Sold
#### So where to start?

1. [The Rust Programming language book](https://doc.rust-lang.org/stable/book/)
2. [Rust By Example](https://doc.rust-lang.org/stable/rust-by-example/)
3. [Rustlings](https://github.com/rust-lang/rustlings)

notes:
So hopefully I have convinced you, Now you may be asking where do I start

1st. Core
2nd is Supplementary 
3rd Interactive examples

---
### Questions

- link to github for slides
- link to github for markdown doc




