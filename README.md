# Lab 1

## Players, Hands, and Games

This lab gives you more hands-on practice with procedural, functional, object-oriented, and generic programming styles in C++17. You will continue extending your card-game abstractions for Pinochle and Texas hold 'em Poker.

In this lab, you will refactor your earlier deck abstractions, introduce abstractions for games and hands, and add a factory function that instantiates game objects polymorphically based on the command line.

Later labs build on these same abstractions, so this assignment is also intended to set up a reusable design for future work.

## Reference

Useful references for this lab:

- C++ strings, vectors, arrays, and I/O: Lippman Chapter 3, plus a skim of Chapter 8
- C++ try blocks and exception handling: Lippman Chapter 5.6
- C++ static class member functions: Lippman Chapter 7.6
- C++ templates for dynamic memory management: Lippman Chapter 12.1.1
- C++ class and function templates: Lippman Chapter 16.1
- C++ random number generators: Lippman Chapter 17.4
- C++ reordering operations: Lippman Appendix A.2.6
- [C++ Reference](https://www.cppreference.com)

## Programming Guidelines

Review the [Programming Guidelines](docs/programming-guidelines.md) before you begin implementing the lab, and keep them in mind as you develop and test your solution.

## Assignment

Record your observations, design decisions, compile warnings or errors, and any other written responses in `ANSWERS.md` as you work.

Some parts of this lab are intentionally somewhat open-ended. When details are under-specified, choose a reasonable design and document that choice clearly in `ANSWERS.md` and comments where helpful.

1. Log into one of the Linux Lab machines via `qlogin`, and confirm that the correct version of `g++` (`8.3.0`) is installed in your environment there, as you did in [Studio 0](https://github.com/cse4208-wustl/studio0).

1. Clone your `lab1` repo and work inside that cloned directory.

1. Use `ANSWERS.md` to record your observations, design decisions, and implementation notes as you develop your solution.

1. Copy over the relevant `.cpp` and `.h` files from your previous lab assignment into this repo as the starting point for Lab 1.

   The provided `Makefile` is already included. If appropriate, you may want to rename files and update the `Makefile` accordingly, for example so that `lab0.cpp` becomes `lab1.cpp`.

1. Define a new `CardSet` class template in its own `.h` file, with its template member definitions in a corresponding source file, and then revise your `Deck` type from the previous lab assignment so that instead of being an abstract base class, it becomes a class template parameterized by rank and suit types and derived by public inheritance from that new `CardSet` class template.

   In `CardSet`:

   - add a protected member variable that is a `vector` of cards parameterized with the same rank and suit types
   - move the `print` member function from `Deck` into the public section of `CardSet`
   - change `print` so it is not virtual and takes a second parameter of type `size_t`
   - define `print` so it inserts cards into the provided `ostream`, separated by spaces, with a line break after the requested number of cards

   In the template source file for `CardSet`, you may find an iterator declaration like this useful for iterating over a vector of cards from beginning (`.begin()`) to end (`.end()`):

   ```cpp
   typename vector< Card<R, S> >::iterator
   ```

   Keep the template include pattern consistent with the previous lab:

   - include the template source file from the header within `#ifdef TEMPLATE_HEADERS_INCLUDE_SOURCE`
   - do this for both `Deck` and `CardSet`
   - make sure the `Makefile` provides `-DTEMPLATE_HEADERS_INCLUDE_SOURCE`

   Update `PinochleDeck` and `HoldEmDeck` so they derive from the appropriately parameterized `Deck` template, and remove their now-duplicated `vector` and `print` members.

   At this point, your program should still compile, run, and produce the same output as your previous lab solution.

1. Extend the `Deck` class template with a `shuffle` member function that takes no parameters and returns `void`.

   It should:

   - declare a device callable object such as `std::random_device`
   - declare a random number generator such as `std::mt19937`, initialized from that device
   - call `std::shuffle` from `<algorithm>` using the deck's underlying vector iterators and the generator

   If the compiler has trouble deducing iterator types, you may need to explicitly scope the vector member variable, for example `CardSet<R, S>::card_vec.begin()`.

1. Add a public right shift operator `operator>>` to `CardSet`.

   It should:

   - take a non-const reference to another `CardSet` with the same type parameters
   - throw an exception such as `runtime_error` if the source card set is empty
   - otherwise copy the card at the back of the source vector into the destination vector
   - pop that card from the source vector
   - return `*this`

   Also add a public `is_empty` member function that returns `true` when the vector is empty and `false` otherwise.

   Use `try` and `catch` appropriately so exceptions are handled promptly and do not propagate uncaught out of `main`.

1. In the header and source files for `Deck`, declare and define a public `collect` member function that takes a reference to a `CardSet` with the same rank and suit types and returns `void`.

   That function should safely move all cards from the passed card set back into the deck.

1. Add a new header file declaring an abstract `Game` base class.

   It should contain:

   - a protected `vector<string>` member variable for player names
   - a public constructor that takes an integer and an array of C-style strings
   - a public pure virtual `play` member function that takes no arguments and returns `int`

1. Add a new source file defining the `Game` constructor.

   It should insert all player names into the vector of strings.

   Since the original `argc` and `argv` also include the executable name and the game name, choose and document a consistent convention for whether:

   - the constructor skips the first two arguments itself, or
   - earlier code adjusts `argc` and `argv` before calling the constructor

1. Add a new header and source file declaring and defining a `PinochleGame` class derived publicly from `Game`.

   `PinochleGame` should have:

   - a protected `PinochleDeck` member variable
   - a protected `vector<CardSet<...>>` member variable storing player hands
   - a protected virtual `deal` member function returning `void`
   - a public constructor taking an integer and an array of C-style strings
   - a public virtual `play` member function returning `int`

   The constructor should:

   - pass its parameters to the `Game` base constructor
   - push back as many hands as there are players
   - calculate that hand count from the constructor arguments rather than hard-coding it

   The `deal` member function should repeatedly shift three-card packets from the deck to each player's hand in turn until all cards are distributed.

   The `play` member function should repeatedly:

   - shuffle the deck
   - deal the cards
   - print each player's name and hand
   - collect all cards back into the deck
   - ask the user whether to end the game
   - read a response from standard input
   - return success if the response is `"yes"`

   Implement this modularly by breaking the logic into helper functions where appropriate.

1. Add a new header and source file declaring:

   - an `enum class HoldEmState` with monotonically increasing values `preflop`, `flop`, `turn`, `river`, and `undefined`
   - a `HoldEmGame` class derived publicly from `Game`

   `HoldEmGame` should have:

   - a protected `HoldEmState` member variable
   - a protected `HoldEmDeck` member variable
   - a protected `vector<CardSet<...>>` member variable storing player hands
   - a protected `CardSet<...>` member variable storing the shared board
   - a public constructor taking an integer and an array of C-style strings
   - a protected virtual `deal` member function returning `void`
   - a public virtual `play` member function returning `int`

   The constructor should:

   - pass its parameters to the `Game` base constructor
   - initialize the state to `HoldEmState::preflop`
   - push back as many hands as there are players

   The `deal` member function should:

   - in `preflop`, deal one card to each player and then a second card to each player, then advance to `flop`
   - in `flop`, deal three cards to the board, then advance to `turn`
   - in `turn`, deal one card to the board, then advance to `river`
   - in `river`, deal one card to the board, then advance to `undefined`
   - in `undefined`, do nothing

   The `play` member function should repeatedly:

   - shuffle the deck
   - reset the state to `preflop`
   - deal the players' hands
   - print each player's name and hand
   - deal and print the flop
   - deal and print the turn
   - deal and print the river
   - collect all cards from the players' hands back into the deck
   - collect the board back into the deck
   - ask the user whether to end the game
   - read a response from standard input
   - return success if the response is `"yes"`

1. In the source file containing `main`, define a `create` function that takes `argc` and `argv` and returns `shared_ptr<Game>`.

   The function should:

   - declare a local `shared_ptr<Game>` that initially points to nothing
   - if the game name is `"Pinochle"`, assign it `make_shared<PinochleGame>(argc, argv)`
   - if the game name is `"HoldEm"`, assign it `make_shared<HoldEmGame>(argc, argv)`
   - return the pointer

1. Modify `main` so it takes parameters of the form `int argc, const char *argv[]`.

   It should:

   - validate the command line
   - accept either `"Pinochle"` followed by exactly four player names, or `"HoldEm"` followed by between two and nine player names
   - print a helpful usage message and return a non-zero error value if the command line is malformed
   - otherwise call `create(argc, argv)`
   - if the returned shared pointer is valid, call `play` and return its result
   - otherwise print an error message and return a non-zero value

1. Run `make` and fix any errors or warnings that occur. In `ANSWERS.md`, record the kinds of errors or warnings you encountered, or note explicitly if you did not encounter any.

1. Run the executable repeatedly and verify:

   - malformed command lines produce helpful errors and non-zero return values
   - the correct number of cards is dealt to the correct number of players in the correct order
   - the Hold 'em board is dealt correctly
   - shuffled deck order differs across separate runs
   - subsequent turns within a run also produce different orderings

1. Update `ANSWERS.md` with:

   - your names
   - your email addresses
   - the lab number
   - whether you encountered warnings or errors while developing the solution
   - what the executable did for each trial you ran

## Deliverables

Commit and push all modified and added files to the repo.
