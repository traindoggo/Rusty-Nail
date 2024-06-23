# Learn Rust with Entirely too many linked lists

I fairly frequently get asked how to implement a linked list in Rust.
The answer honestly depends on what your requirements are,
and it's obviously not super easy to answer the question on the spot.

As such I've decided to write the book to comprehensively answer the
question once and for all.

```
私はかなり頻繁に RustでのLinked Listの実装方法について聞かれてきました
正直に言うと、この答えはあなたが何を求めているのかに強く依存しており
なのでこの場で答えを出すのは全く簡単ではありません

というわけで、私はこれっきりにするためにこの答えに完全に答えるために
この本を書くことを決めました
```

In this series I will teach you basic and advanced Rust programming entirely
by having you implement 6 linked lists. In doing so, you should learn:

- The following pointer types: &, &mut, Box, Rc, Arc, *const, *mut, NonNull(?)
- Ownership, borrowing, inherited mutability, interior mutability, Copy
- All The Keywords: struct, enum, fn, pub, impl, use, ...
- Pattern matching, generics, destructors
- Testing, installing new toolchains, using miri
- Unsafe Rust: raw pointers, aliasing, stacked borrows, UnsafeCell, variance

```
この本ではRust言語の基礎から応用までをすべて教えることになります
6つのLinked Listを実装することで
そのためにあなた以下を学ぶ必要があります
```

Yes, linked lists are so truly awful that you deal with all of these concepts
in making them real.

```
ね？linked list は実際にひどいです
Linked Listを実際に実装するためには
あなたはこれらの概念をすべて扱えるようになる
```

Everything's in the sidebar (may be collapsed on mobile), but for quick reference,
here's what we're going to be making:

- A Bad Singly-Linked Stack
- An Ok Singly-Linked Stack
- A Persistent Singly-Linked Stack
- A Bad But Safe Doubly-Linked Deque
- An Unsafe Singly-Linked Queue
- TODO: An Ok Unsafe Doubly-Linked Deque
- Bonus: A Bunch of Silly Lists

Just so we're all the same page, i'll be writing out all the commands that I feed
into my terminal.

```
ちょうど同じページにRustのコマンド書いときます
```

I'll also be using Rust's standard package manager, Cargo, to
develop the project. Cargo isn't necessary to write a Rust program, but it's so
much better that using rustc directly. If you just want to futz around you can
also run some simple programs in the browser via `play.rust-lang.org`.

```
そして Rust標準のパッケージマネージャーの Cargo を利用します
Cargoは絶対に必要というわけではありませんが
rustc を直接使うよりも遥かに有用です
ただプログラムをいじくり回したいだけなら、ブラウザ上(...)で簡単なプログラムを
実行することもできる
```

In later sections, we'll be using "rustup" to install extra Rust tooling.
I strongly recommned intalling all of your Rust tool-chains using rustup.

```
後のセクションでは rustupという別のRustツール使います
rustupもインストールしておくこと、おすすめです
```

Let's get started and make our project.

```
> cargo new --lib lists
> cd lists
```

We'll put each list in a separate file so that we don't lose any of our work.

```
同じファイルにLinked Listを書いていくので
忘れること、失うことは無いでしょう
```

It should be noted that the authentic Rust learning experience involves writing
code, having the compiler scream at you, and trying to figure out what the heck
that means.

```
これは注目されるべきことですが、正真正銘のRustのプログラミング体験とは
コードを書いて、コンパイラーはそれについて大声て指摘をします
で、それが何を意味しているのかを理解しようと試みることなのです
```

I will be carefully ensuring that this occurs as frequently as possible.
Learning to read and understand Rust's generally excellent compiler errors and
documentation is incredibly important to being a productive Rust programmer.

```
私はこのようなことができるだけ頻繁に起こるように注意深くしてます
Rust の素晴らしいコンパイラエラー、そしてドキュメントをを読んで理解、
学ぶことこそが、生産的なRustプログラマーになるうえで最重要項目の一つです
```

Although actually that's a lie. In writing this I encountered way more compiler
errors than I show. In particular, in the later chapters I won't be showing a lot
of the random "I typed (copy-pasted) bad" errors that you expect to encounter in
every language. This is a guided tour of having the compiler scream at us.

```
しかしながら実際には嘘です(?) コードを書いていく上で、あなたが見るよりも多くの
コンパイルエラーに出会うでしょう 特にこの後の章ではどの言語でもであるであろう
大量のコピペコードエラーを見せるつもりはありません
コンパイラが我々に向かって叫び倒すガイドツアーなのです
```

We're going to be going pretty slow, and I'm honestly not going to be very
serious pretty much the entire time. I think programming should be fun, dang it!

```
我々の進行度合いはとても遅くなるでしょうが、全体の進捗として
そこまで深刻な低速度では無いと信じたい
プログラミングは楽しいんでね
```

If you're the type of person who wants maximally information-dense, serious,
and formal content, this book is not for you.
Nothing I will ever make is for you. You are wrong.

```
もしあなたがコスパ至上主義だったり、論文チックな本をお好みだったら
この本はあなた向きではありありません
私がこれから作るものは何一つあなたのためのものではありません
```

## An Obligatory Public Service Announcement

Just so we're totally 100% clear: I hate linked lists. With a passion.
Linked lists are terrible data structures.
Now of course there's several great use cases for a linked list:

- You want to do a lot of splitting or merging of big lists. A lot.
- You're doing some awesome lock-free concurrent thing.
- You're writing a kernel/embedded thing and want to use an intrusive list.
- You're using a pure functional language and the limited semantics
  and absence of mutation makes linked lists easier to work with.

But all of these cases are super rare for anyone writing a Rust program.
99% of the time you should just use a Vec (array stack), and 99% of
the other 1% of the time you should be using a VecDeque (array deque).

```
ただ、Rustを書いている誰であってもこれらすべてのケースはとてもレアなものです
99%の場合は vector(array stack) で事足りるし
残りの1% であっても deque でよいでしょう
```

These are blatantly superior data structures for most workloads due to less
frequent allocation, lower memory overhead, true random access,
and cache locality.

```
これらの優れたデータ構造は明らかに、以下の理由において
ほとんどのワークロードにおいて明らかに優れている
メモリのオーバーヘッドが少ないこと、ランダムアクセスができる、
キャッシュの局所性
```

Linked lists are as niche and vague of a data structure as a trie.
Few would balk at me claiming a trie is a niche structure that a niche
structure that your average programmer could happily never learn in an entire
productive carrer -- and yet linked lists have some bizarre celebrity status.

```
Linked list はトライと同じくらいニッチで曖昧なデータ構造です
トライはだいぶニッチなデータ構造で、幸せなことに平均的なプログラマーが
生産的なキャリアの中で学ぶことはないでしょう
```

We teach every undergrad how to write a linked list.
It's the only niche collection (...).

```
我々はあらゆる学部生にLinked Listをどうやって書くのかを教えます
```

We should all as a community say no to linked lists as a "standard" data
structure. It's a fine data structure with several great use cases, but those
use cases are exceptional, not common.

```
我々は(すべてのコミュニティとして?) LLは標準的なデータ構造では無いと言うべきです
原点的なユースケースであれば良いデータ構造ですが
これらの使用例は例外的なものであり、一般的なものではありません
```

Several people apparently read the first paragraph of this PSA
and then stop reading. Like, literally they'll try to rebut my argument
by listing one of the things in my list of great use cases.
The thing right after the first paragraph!

```
この最初の段落で、読むやつが何人かいるようだ
文字通り彼らは私の素晴らしい使用例のリストの一つを上げて
私の議論に反論しようとするのだ
```

Just so I can link directly to a detailed argument,
here are several attempts at counter-arguments I have seen,
and my response to them. Feel free to skip to the first chapter
if you just want to learn some Rust!

## Performance doesn't always matter

Yes! Maybe your application is I/O-bound or the code in question
is in some cold case that just doesn't matter.
But this isn't even an argument for using a linked list.
This is an argument for using whatever at all.
Why settle for a linked list? Use a linked hash map!

```
あなたのアプリは I/Oバウンド してるかもしれないし 問題のコードはどうでもいい
ような例外的なケースなのかもしれない
しかしこれらはLLをつ開くための議論にすらならない
これはどんなものでも使うべきなのだという主張で
どうしてLLなんて使うの? LL hashmap を使えばいいじゃん!
```

If performance doesn't matter,
then it's surely fine to apply the natural default of an array.

```
パフォーマンスが関係ないのであれば 普通の配列でも使えばいいし
```
