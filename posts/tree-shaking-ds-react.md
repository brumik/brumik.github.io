---
title: Tree Shaking in our Design System Library
date: 2025-01-31
permalink: /blog/2025/01/31/tree-shaking-in-our-ds-lib
---

We are developing with the best effort our nice not-so little design component library. It is like the big ones, but smaller. When you do a common library for multiple projects you encounter many new opportunities to learn. You have to set up `beta` releases, you need to get guidelines in place and of course you need to optimize it.

## The trigger for optimization

Every programmer probably knows, that premature optimization is best avoided. However, when you realize that on page load you just download all your icons in a JSON format, no matter if you need it or not, you know that you *need* to do something.

Looking into it also showed that this is caused by our commonly share design system. This is a much bigger issue, since all of our frontends will suffer from the same issue.

## Original implementation

When migrating components into the new and fancy design system, we just copied the components we needed. The Icon components was fairly simple. It went something like this:

```js
import Icons from 'icons.json';
const Icon = ({ name }) => 
    <span dangerouslySetHTML={ Icons[name] } /> 
```

You can see the problem right away. All the `svg` is stored in a single `icons.json` file. This is really flexible since you can control the icon with a prop, but also means you need to download all icons if you use any of them.

And we had many of them, exactly 600kB of them. In perspective that is 4x amount the recommended initial bundle size, and it is half the size our complete bundle.

## Solution

After a bit of thinking I realized that we cannot keep the interface. There is a reason that when you import icons from other libraries it always goes like `import { MyIcon } from 'good-icons`. 

So what I did was to `scriptify` the whole process and split up each icon into its own component. Having an icon as a standalone component that is not dependent on any crazy imports is important, since then it can be split up and pulled into bundles individually.

The end result was consisting of 4 parts:
- A folder with pure `.svg` files that are used as a source
- A folder with pure `IconName.tsx` that are the generated icon components with inline `svg` part
- A script that takes all the `.svg` and creates the components.
- A simple `Template.tsx` file to serve as a template.

The template file was relatively simple:

```tsx
export const __SVG_COMPONENT_NAME__ = (props) => 
    <span dangerouslySetHTML=`__SVG_DATA__` ...props />;

```

This is of course oversimplified (and the dangerous part was removed later) but as you can see, I just wrote some incorrect `tsx` with some `string` placeholders.

Then in the `node` script I kinda do the following:

```js
// To run this srcipt:
// node generateIcons.js iconSvgPath iconTsxPath iconTemplatePath.tsx
import { basename, join } from 'path';
import { promises as fs } from 'fs';

function toPascalCase(str) {
  return str
    .replace(/[-_](.)/g, (_, char) => char.toUpperCase())
    .replace(/^(.)/, char => char.toUpperCase());
}

// Get command-line arguments
const args = process.argv.slice(2);
const [dirPath, outPath, templateFilePath, suffix] = args;

if (!dirPath || !outPath || !templateFilePath) {
  console.error("Usage: node script.js <dirPath> <outPath> <templateFilePath> <suffix>");
  process.exit(1);
}

async function processFiles() {
  try {
    const template = await fs.readFile(templateFilePath, 'utf-8');
    const exports = [];

    const files = await fs.readdir(dirPath, { withFileTypes: true });
    for (const file of files) {
      if (file.isFile() && file.name.endsWith('.svg')) {
        const filePath = join(dirPath, file.name);
        const svgContent = await fs.readFile(filePath, 'utf-8');

        const svgName = basename(file.name, '.svg');
        const pascalCaseName = toPascalCase(svgName) + suffix;

        const resultContent = template
          .replace(/__SVG_NAME__/g, svgName)
          .replace(/__SVG_COMPONENT_NAME__/g, pascalCaseName)
          .replace(/__SVG_DATA__/g, svgContent);

        const outputFilePath = join(outPath, `${pascalCaseName}.tsx`);
        await fs.writeFile(outputFilePath, resultContent);
        console.log(`Written to file: ${outputFilePath}`);

        exports.push(`export { ${pascalCaseName} } from './${pascalCaseName}';`);
      }
    }

    // Write all exports to index.ts
    const indexPath = join(outPath, 'index.ts');
    await fs.writeFile(indexPath, exports.sort().join('\n'));
    console.log(`All exports written to index.ts`);

  } catch (error) {
    console.error("An error occurred:", error);
  }
}

processFiles();
```

In short:
	- Iterate through the `svg` folder and for each icon
		- create a new file using the `Template` and replace strings like `__SVG_NAME__` with correct values generating valid `tsx` files.
	- Create an `index.ts` file which exports all the generated components

The arguments are there because we started to use this script to generate the `Illustration` components too. 

## Summary

- We reduced the initial bundle size from 1MB to 150kB while preserving all the functionality.
- We did introduce a breaking change to the library but all the migration is just a boring hour of manual work

Is this the best possible solution? Probably not, but for the project size and the allocated time this gets the work done and fixing our bundle size issue.
