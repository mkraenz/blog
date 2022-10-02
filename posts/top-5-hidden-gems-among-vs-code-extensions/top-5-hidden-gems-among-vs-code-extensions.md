---
title: Top 5 Hidden Gems among VS Code Extensions
description: TODO
tags: 'typescript, javascript, vscode'
cover_image: 'https://res.cloudinary.com/practicaldev/image/fetch/s--D3t6KdBd--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3yzspddr8hbaq6pksosa.png'
published: false
id: 709305
---

Visual Studio Code is a great editor in itself. Paired with the power of plugins and extension it transforms into heaven for every developer. Today, I'll share with you my Top 5 not-so-well-known plugins which I'm sure will make your life a lot easier.

## tl;dr

- [Anki for VSCode](https://marketplace.visualstudio.com/items?itemName=jasew.anki) - flashcard creation from markdown with code snippet support
- [Diff Folders](https://marketplace.visualstudio.com/items?itemName=L13RARY.l13-diff) - git diff on folders
- [Partial Diff](https://marketplace.visualstudio.com/items?itemName=ryu1kn.partial-diff) - git diff your clipboard and selected text
- [Bracketeer](https://marketplace.visualstudio.com/items?itemName=pustelto.bracketeer) - change brackets and quotes with ease
- [Copy Relative Path and Line Numbers](https://marketplace.visualstudio.com/items?itemName=ezforo.copy-relative-path-and-line-numbers) - `Alt + L` to copy

## Anki for VSCode

If you've tried learning some new language, you probably know [Anki](https://apps.ankiweb.net/), a flashcard app which comes also with a desktop app for easy editing of flashcards. Having spend more than 2000 days with mobile and desktop version in total, I can tell you that flashcards for coding are possible but not really comfortable to write.

Then came [Anki for VSCode](https://marketplace.visualstudio.com/items?itemName=jasew.anki)! All I write now is regular Markdown and it gets send to Anki without me touching my computer mouse! Love it. In particular, Markdown gets formatted on save thanks to another VS Code plugin ([markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint)).
I only open the Anki Desktop App for 2 seconds when I want to send my flashcards in markdown to Anki. Here's an example flashcard with front `ts union type` and backside `number | string` which gets send to the `Informatics` deck.

````markdown
# Informatics

## ts union type

```ts
type MyUnion = number | string;
```

## Diff Folders

If you ever had to migrate packages out of a mono repo into their own repo (or vice-versa) while other developers were still working on the existing mono repo package, well, then you probably know the pain of having to manually sync 50 or more files from A to B.

[Diff Folders](https://marketplace.visualstudio.com/items?itemName=L13RARY.l13-diff) made this process a lot easier by allowing to git-compare complete folder structures with eachother. Fortunately, this such a migration is not an everyday task but if it comes you want to have Diff Folders installed.

## Partial Diff

Similar to Diff Folders but more on the everyday basis. My favorite part of [Partial Diff](https://marketplace.visualstudio.com/items?itemName=ryu1kn.partial-diff) is comparison with clipboard. Last time I used this I compared two jest console outputs with eachother without having to save the logfiles (thereby avoiding the chore of cleanup 😉). After copying one output to clipboard, you can mark the other output and right-click -> `Compare Text with Clipboard`.

## Bracketeer

Changing _brackets and quotes_ into a different style can be annoying. [Bracketeer](https://marketplace.visualstudio.com/items?itemName=pustelto.bracketeer) makes it less annoying so you can focus on the code. Bracketeer can replace or delete all important quotes (backtick, single and double quotes) and parentheses.  
Let's say you find some code saying `drink("coffee")` but you want a [template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) instead like so

```typescript
const drinkA = "tea";
const drinkB = "coffee";
drink(`${drinkA} and ${drinkB}`);
```

Of course, you can do it manually but that quickly becomes a chore. With Bracketeer I simply put my cursor somewhere inside of `"coffee"` and hit `Shift+Cmd+Alt+;`. Then I have my backticks. If you find the shortcut too complicated, bracketeer allows for rebinding all keys which I always find is a good thing.

## Copy Relative Path and Line Numbers

Sometimes, simple things are also the best. For communication, I often want to quickly and easily send a specific line in code with it's position to a team member. That's exactly where [Copy Relative Path and Line Numbers](https://marketplace.visualstudio.com/items?itemName=ezforo.copy-relative-path-and-line-numbers) shines. One hit on `Alt + L` and wabams, I have the relative path `src/App.tsx:16` in the clipboard. Particularly nice is that my team member can copy-and-paste it into VS Code's `Go to File` search (i.e. `Ctrl + P`) and the cursor lands exactly in the right place.

## Closing

Wanna learn more about VS Code? Hit subscribe on the blog, and join our [Twitch TypeScriptTeatime](https://www.twitch.tv/typescriptteatime/schedule) for Live Coding Fun! Looking forward to see you!

## Recommended Reading

[BaldBeardedBuilder](https://www.twitch.tv/baldbeardedbuilder) has an amazing article [10 VS Code Extensions You Need Today](https://michaeljolley.com/blog/10-visual-studio-code-extensions-you-need-today/). I already used 7 out of 10 so the article has my fullest support. Check it out and spread some developer love. 💚
````
