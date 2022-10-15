# YAML

YAML is a "human-friendly" format to succeed data. YAML stands for "Yet Another Markup Language." You can recognize a YAML file by
the extension `.yaml` or `.yml`. Why do we see YAML? Our DevOps tools - especially Kubernetes - are going to use code and data to build and manage our infrastructure.
Many tools, almost all of them, chose to do this in YAML. YAML aims to be easily readable but also writable. For this, they borrowed inspiration from Python.

YAML is a superset on JSON. We can convert YAML to JSON and JSON data is just valid YAML data.

This is an example of YAML (written by AI, cool huh):

```yaml
---
# This is a comment
name: John Doe
age: 43
address:
  street: 123 Main St.
  city: Anytown
  state: NY
  zip: 12345
children:
  - John
  - John
  - Jack
pets: ["dog", "cat", "bird"]
```

## YAML compoments

A YAML document starts with `---` but this is optional, it is useful when you want to put several YAML documents in one file.
In YAML we often encounter `key: value`. With this we assign a value to a key. This can be

- a string, just write the text
- a number, just write the number
- an array, we write it with a new line followed by a `-` or the values between `[]`.
- an object, we write it with a new line followed by keys and values again

## Writing YAML

Okay, let's be honest for a moment YAML has its negative sides. YAML mandates spaces for indentation, no tabs allowed! So it is not always easy to find an error against it since it is an invisible character. For example, some have figured out to just use a ruler....
![a yaml ruler](./yaml-ruler-cover.jpg)

We prefer to solve this with good tooling. The [Visual Studio Code](https://code.visualstudio.com/) code editor (not to be confused with the Visual Studio IDE) is a handy tool to write YAML as well as apply our version control and write code. It also helps us with alignment, also there are several plugins available to help with YAML and other tools in this course.

### Tips

#### Linter

A linter (a static code analysis tool used to flag programming errors, bugs, stylistic errors and suspicious constructs) can help you detect errors in YAML. `yamllint` is a widely used CLI tool that you can install in any Linux distribution. There are also web versions like [YAML Linter](https://yamllint.com/).

#### Tabs and Spaces

A tab or 4 spaces can look the same, if you copy code it can sometimes go wrong here. If you highlight the text in VS Code it will show you the spaces and tabs with symbols. By default, VS Code will use 4 spaces when you use the tab key. You can modify this behavior using "Spaces" button at the bottom of the VS Code windows. Then a menu appears where you can choose how many spaces, or the size of the tab, that should be used.
![conversion menu](./convert.png)

As already mentioned, YAML forces you to use spaces in this scenario. [I'll stop there, tabs and spaces often cause too much discussion among ITers](https://www.youtube.com/watch?v=cowtgmZuai0).
