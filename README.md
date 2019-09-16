# Redis documentation

## Clients 客户端

All clients are listed in the `clients.json` file.
`clients.json` 文件列出了所有的客户端。
Each key in the JSON object represents a single client library.
JSON对象的每个key代表着单个客户端库。
For example:
例如：

```
"Rediska": {

  # A programming language should be specified.
  # 应指明使用的编程语言。
  "language": "PHP",

  # If the project has a website of its own, put it here.
  # 如果项目有其网站，在此列出。
  # Otherwise, lose the "url" key.
  # 没有则不要列出“url”这项。
  "url": "http://rediska.geometria-lab.net",

  # A URL pointing to the repository where users can
  # find the code.
  # 一个指向源代码库的URL。
  "repository": "http://github.com/Shumkov/Rediska",

  # A short, free-text description of the client.
  # Should be objective. The goal is to help users
  # choose the correct client they need.
  # 一段简短的、自由发挥的客户端描述。
  # 应保持客观。目的是为了帮助用户选择他们需要的客户端。
  "description": "A PHP client",

  # An array of Twitter usernames for the authors
  # and maintainers of the library.
  # 一个数组，其中为库的作者和维护者的Twitter用户名。
  "authors": ["shumkov"]

}
```

## Commands

Redis commands are described in the `commands.json` file.

For each command there's a Markdown file with a complete, human-readable
description.
We process this Markdown to provide a better experience, so some things to take
into account:

*   Inside text, all commands should be written in all caps, in between
    backticks.
    For example: `INCR`.

*   You can use some magic keywords to name common elements in Redis.
    For example: `@multi-bulk-reply`.
    These keywords will get expanded and auto-linked to relevant parts of the
    documentation.

There should be at least two predefined sections: description and return value.
The return value section is marked using the @return keyword:

```
Returns all keys matching the given pattern.

@return

@multi-bulk-reply: all the keys that matched the pattern.
```

## Styling guidelines

Please use the following formatting rules:

* Wrap lines to 80 characters.
* Start every sentence on a new line.

Luckily, this repository comes with an automated Markdown formatter.
To only reformat the files you have modified, first stage them using `git add`
(this makes sure that your changes won't be lost in case of an error), then run
the formatter:

```
$ rake format:cached
```

The formatter has the following dependencies:

* Redcarpet
* Nokogiri
* The `par` tool

Installation of the Ruby gems:

```
gem install redcarpet nokogiri
```

Installation of par (OSX):

```
brew install par
```

Installation of par (Ubuntu):

```
sudo apt-get install par
```

## Checking your work

You should check your changes using Make:

```
$ make
```

This will make sure that JSON and Markdown files compile and that all
text files have no typos.

You need to install a few Ruby gems and [Aspell][aspell] to run these checks.
The gems are listed in the `.gems` file. Install them with the
following command:

```
$ gem install $(sed -e 's/ -v /:/' .gems)
```

The spell checking exceptions should be added to `./wordlist`.

[aspell]: http://aspell.net/
