# Czara Minecraft Seed Generator

## Purpose of Program

This small program was written to generate 64-bit signed integer seeds
for Minecraft based on arbitrary UTF-8 input (Can be expanded in the
future).

## Reason for Project
I was setting up a Minecraft Server for myself and my friends, and as I was looking through the server configuration file, I saw that the seed could be any 64-bit integer or **string/phrase**.  Curious about if this was true, I decided to search light heartedly on Google about how this worked, and it turns out any **string/phrase** entered as a seed goes through a default Java `String.hashCode()` function which returns only a 32-bit integer.

I looked at this information and thought, why are strings and phrases only 32-bit integers versus what Minecraft can actually utilize, so I decided to take a stab at generating 64-bit integers using user input.  I didn't want to just use a standard hashing algorithm, so I looked up how to do hashing in Python.  Though I don't normally use it myself, the AI Generated Answer from Google provided me a good baseline for making different hashes with different algorithms.

I let my hands spend another hour of coding and two hours of debug, and I was finished with this.

## How to use
Project is installed using `python -m pip install .` where `.` is the project's directory with the `pyproject.toml`.  This installs the software on your machine where python is installed.

The command installed is `mcseedgen` to execute.

The command has 3 arguments:
 - `-h` or `--help` (Opens help message)
 - `-i` or `--init` (The argument that informs the program you are entering the initializer)
 - `-a` or `--algorithm` (Hash Algorithm to generate seed with the phrase)

I recommend using `-h` to actually read the arguments.

Example: `mcseedgen -i HelloWorld -a sha512`
```
PS D:\my\path> mcseedgen -i HelloWorld -a sha512
        >> Generated random seed (64-bit signed int) is -6828754803009022345
```

The project also logs the generated seeds in a JSON file: `czara_seed_gen.history.json`

Example (Continued...):
```
{
    "HelloWorld": {
        "sha512": {
            "hash": "8ae6ae71a75d3fb2e0225deeb004faf95d816a0a58093eb4cb5a3aa0f197050d7a4dc0a2d5c6fbae5fb5b0d536a0a9e6b686369fa57a027687c3630321547596",
            "hash-decimal": 7274840862189094729519536349162555926467336013579474867256175737922050493109128770798841505446376269147667595390707919438543975953833909450475148049413526,
            "seeds": [
                -6828754803009022345
            ]
        }
    }
}
```

## How it works
1. Takes user input or utilizes current nanosecond time since epoch.
1. Generates hash using provided algorithm (md5 by default if not provided)
1. Converts hash to decimal and seeds Python's built-in Random library with it.
1. Checks if JSON history exists, and if initializer and algorithm are present in it.
   1. If they are present, it gets the number of seeds previously generated with them.
1. It will generate *N* keys where *N* is the number of previous entries if any.
   - This should allow the algorithm to not produce a duplicate entry without any clashes that occurred from Hash Algorithm and Random Number Algorithm.
   - For more information on why I am doing this, please look into the following (I'll provide my thoughts later if you still want to know):
      - Pseudorandom Number Generation
      - Seeding Random Number Generators
1. Generate the current 64-bit integer to be displayed.
   - Number is unsigned using `random.getrandbits(64)`, so I subtract `2^63 (2**63)` from it to make it signed.
1. Update JSON History Log
1. Print result to user.

## Reason for design

I wanted to allow users to source their MC seed using a phrase, but you can not normally seed random number generators from strings, so I wanted to use the string to generate a number that can be used as a seed to the number generator.  This is the reason why I am generating a Hash from the string using a hasing algorithm.  The Hashed string is a large **Hexadecimal** integer.  I use Python's method for converting to **Decimal** (base-10).  I then seed Python's [Random](https://docs.python.org/3/library/random.html) library with that Hash value.  Since, the first entry expected back from a Seeded **"Random"** Number will always be the same as well as the next *N* results from calling the same Python `getrandbits` function, I filter through those *N* results, that way, we get the next **random** number for your phrase.  This way you can generate numerous seeds from your single phrase.  If you want a previous phrase, they are recorded in the generator's log JSON file, `czara_seed_gen.history.json`.

Now, is this fullproof? **No**, as with all Hashing Algorithms and **Pseudo**random number generation, you will see collisions at some point.  The goal is to minimize the collisions as much as possible by going through multiple steps to generate the number.  If you want to explore solutions that don't rely on *Pseudo*random number generation, then please check out [Lavarand](https://en.wikipedia.org/wiki/Lavarand).
