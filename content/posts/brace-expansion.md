---
title: 'Brace Expansion'
tags: [core utils, bash]
date: '2024-12-01'
draft: false
---

If you interact with Bash on a regular basis, chances are you're familiar with [Filename Expansion][1]. For example:

```bash
find . -type f -name "*.md"
```

{{% comment %}} TODO: turn this into a shortcode {{% /comment %}}
{{< rawhtml >}}<sup><em>Find all files in the current directory that end in ".md"</em></sup>{{< /rawhtml >}}

The `*.md` pattern will expand all the filenames that match:

```bash
./README.md
./wiki/home.md
```

However, you may have never heard of [Brace Expansion][2]. Brace expansion falls under the umbrella of [Shell Expansions][3], just like filename expansion.

## Example 1

```bash
touch {aa,ab,ac}.txt
```

The result:

```bash
❯ ls -1

aa.txt
ab.txt
ac.txt
```

I've been able to use brace expansion with any command that takes a path as an argument. If I know I need to run the same command on several files and/or directories, brace expansion allows me to run that command once instead of `n` times, where `n` is the number of files plus directories.

## Example 2

I'm in a project that has hundreds of thousands of files. I need to find `numa-numa.mp4`. I know that it is either in `assets/` or `public/`. Given that, I can run the following:

```bash
find {assets,public} -type f -name "numa-numa.mp4"
```

The alternatives to using brace expansion for this example would either be using `find .` or running the `find` command twice: once for `assets/` and once for `public/`. Using `find .` will search everywhere in the current directory and will eventually locate the file. However, this will be significantly slower as there are many files to look through. Running `find` twice is just annoying and gets *really* painful when we start talking about ten directories as opposed to two.

## Example 3

This example represents a real use case I have encountered in my career. I needed to copy certain files and directories from a mounted volume to another location on a server. However, I did *not* want to copy certain other files/directories from the volume. To accomplish this, I ran a command that ended up looking similar to:

```bash
rsync -aP /mnt/uuid/{config.json,assets/,plugins/,data/} /the/target/path/
```

The `assets/` dir in particular was many gigabytes in size and was going to take awhile to transfer. I wanted to fire and forget, not constantly be checking back on its progress to start copying the next directory. To that end, if I had tried doing this in a
{{< rawhtml >}}<span style="cursor:help;text-decoration:underline var(--content) dashed" title="You'll understand the sarcastic quotes in a moment">"one-liner"</span>{{< /rawhtml >}}{{% comment %}} TODO: turn this into a shortcode {{% /comment %}}
before knowing about brace expansion, I probably would have written something akin to:

```bash
find /mnt/uuid -maxdepth 1 \
  -name config.json -o \
  -name assets -o \
  -name plugins -o \
  -name data \
  -exec rsync -aP {} /the/target/path/ +
```

Not only does this require more typing (boo :thumbsdown:), it also uses syntax of `find` that I don't use often, meaning I'm more likely to make a mistake.

While writing this post, I discovered that brace expansions can be nested within each other or even be combined with [Pattern Matching][4]:

```bash
chown root /usr/{ucb/{ex,edit},lib/{ex?.?*,how_ex}}
```

I don't fully understand what this command is doing, but [it's giving][5] Regex so I'm excited to explore more in the future.

Until then, I'll keep using the basics of brace expansion in my day-to-day for convenience in the spirit of ~~laziness~~ efficiency.

[1]: https://www.gnu.org/software/bash/manual/html_node/Filename-Expansion.html
[2]: https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html
[3]: https://www.gnu.org/software/bash/manual/html_node/Shell-Expansions.html
[4]: https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html
[5]: https://knowyourmeme.com/memes/its-giving
