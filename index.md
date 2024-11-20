<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Agyei Lomini - Portfolio</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            line-height: 1.6;
            color: #333;
            background-color: #ffffff;
        }

        header {
            background: #0056b3;
            color: #ffffff;
            padding: 1rem;
            text-align: center;
        }

        header h1 {
            margin: 0;
        }

        header p {
            margin: 0.5rem 0 0;
        }

        #content {
            display: flex;
            flex-wrap: wrap;
            margin: 0;
        }

        aside {
            width: 25%;
            background: #f4f4f4;
            padding: 1rem;
            box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
            box-sizing: border-box;
            min-height: 100vh;
        }

        aside h2 {
            color: #0056b3;
            margin-top: 0;
        }

        aside ul {
            list-style: none;
            padding: 0;
        }

        aside ul li {
            margin-bottom: 1rem;
        }

        aside ul li a {
            color: #0056b3;
            text-decoration: none;
        }

        aside ul li a:hover {
            text-decoration: underline;
        }

        main {
            flex: 1;
            padding: 2rem;
            box-sizing: border-box;
        }

        section {
            margin-bottom: 2rem;
        }

        section h2 {
            color: #0056b3;
            border-bottom: 2px solid #0056b3;
            padding-bottom: 0.5rem;
        }

        footer {
            text-align: center;
            padding: 1rem 0;
            background: #0056b3;
            color: #ffffff;
            margin-top: 2rem;
        }
    </style>
</head>

<body>
    <header>
        <h1>Agyei Lomini</h1>
        <p>Game Developer | Programmer | Creator</p>
    </header>

    <div id="content">
        <aside>
            <h2>Projects</h2>
            <ul>
                <li><a href="https://github.com/your-repo/game-project" target="_blank">Unity Game - State Machine
                        Adventure</a></li>
                <li><a href="https://github.com/your-repo/dnd-system" target="_blank">Homebrew DnD Campaign System</a>
                </li>
                <li><a href="https://github.com/your-repo/ai-simulation" target="_blank">AI Simulation - Enemy Behavior
                        in Unity</a></li>
            </ul>
            <h2>Resume</h2>
            <p><a href="your-resume-link.pdf" target="_blank">Download Resume (PDF)</a></p>
        </aside>

        <main>
            <section id="about">
                <h2>About Me</h2>
                <p>Hello! I'm Agyei Lomini, a passionate game developer with experience in Unity, game mechanics design,
                    and coding. I love creating immersive experiences and solving technical challenges in game
                    development.</p>
            </section>

            <section id="contact">
                <h2>Contact Me</h2>
                <p>Email: <a href="mailto:your-email@example.com">your-email@example.com</a></p>
                <p>GitHub: <a href="https://github.com/your-username" target="_blank">github.com/your-username</a></p>
                <p>LinkedIn: <a href="https://linkedin.com/in/your-profile"
                        target="_blank">linkedin.com/in/your-profile</a></p>
            </section>
        </main>
    </div>

    <footer>
        <p>&copy; 2024 Agyei Lomini | Built with ❤️ using GitHub Pages</p>
    </footer>
</body>

</html>