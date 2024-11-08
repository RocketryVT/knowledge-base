# Contributing

- Clone the repository
- Or use Github Codespaces to edit in the browser

1. Navigate to the `src` directory.
2. Edit or add markdown files.
3. Update the `SUMMARY.md` file to include new files.
4. Commit your changes.
5. Push your changes to GitHub.
6. GitHub Actions will build and deploy the site.
7. View the updated site on GitHub Pages after a few minutes.

## Using MathJax

The following markdown code will render a math equation using MathJax.

~~~markdown
```mermaid
\begin{align*}
\dot{x} & = \sigma(y-x) \\
\dot{y} & = \rho x - y - xz \\
\dot{z} & = -\beta z + xy
\end{align*}
```
~~~

The output will look like this:
```mermaid
\begin{align*}
\dot{x} & = \sigma(y-x) \\
\dot{y} & = \rho x - y - xz \\
\dot{z} & = -\beta z + xy
\end{align*}
```

## Using Mermaid.js

The following markdown code will render a flowchart using Mermaid.js.

~~~markdown

```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```
~~~

The output will look like this:

```mermaid

graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```

For more information, see the [Mermaid.js documentation](https://mermaid.js.org/).

Use the online editor to test out you graphs here: [Mermaid Live Editor](https://mermaid.live/edit)

An example of more complex Mermaid.js can be found [here](https://mermaid.live/edit#pako:eNqNlV1vmzAUhv8Kcm86iSTGJLRYVaU2VaVNnTStdytVZGwn9QoY2aYtq_rfZyAfkHhZc5GYc948fjnnGN4BlYwDDJaZfKVPRBnv7mdSJMUkJ0VFsoUoyso8eINL7xFjnBPRyJ55nUqi2EJxwriyyr1IKy7sJi3zRUmta90C1-tDgcirrcCuDwU0Z1uBXfcFv2V9anP25yJVl5oXWqpFrld6Yr8m32TtPX6xaiNLQfv22xs77dtvIy3EsB3h3ihRrIaQzkhKjOGq3X0QcPm47lL3hhjuYk3sbTNiyA62ibhoX5sauSHa7iDoolRc60rxIW8v6ULfZpVgP9YC5ya2_lTmdjzYw8Q7DDbQLthBtXqZzLvUnbSFTJKmeZqrF0F5O3i2dd5odDkcuYNWOSWDuh9RbIr5H8lefY67Wk97qxla7Q_-htHz2R_7fnpj8lh-z6GrK54cjUeyd1r2zrZ3YfOXjs41zaAZ0fqGL73mdHmyJFSYGsNx7C9FluETdAV9bZR85vgkjIP1evQqmHnC0_LNpzKTCp9ACHswQo2QhQMXRVdbHEKzIQ79C7ceHpc9xKMw2jm8jmCEPkltx9zB5CmdhumWOZ_PjwF9TUnGcTBGPXTz6HSQw5SFfEeG8e0n65lWqxVnDmIchlvcOYTHjAIf5FxZZ8y-DN6TwvMSYJ54zhOA7ZLxJakyk4Ck-LBSUhl5XxcUYKMq7oOqtJPKbwRZKZJvgpwJI9X37v3SvmZ8UJLil5RWsiSZ5u01wO_gDeDpGEJ0HgYIwWiKgrPYBzXAAYrGcRifzVAcBNEshNMPH_xpCYH9Q_eJZzGM0dnHXy9TZqI)
