![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=Shrooxie&show_icons=true&theme=tokyonight)


![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=Shrooxie&hide_progress=true&show_icons=true&theme=tokyonight)

[![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=Shrooxie&layout=pie&show_icons=true&theme=tokyonight)](https://github.com/Shrooxie/github-readme-stats)


![ChatGPT](https://img.shields.io/badge/chatGPT-74aa9c?style=for-the-badge&logo=openai&logoColor=white)
![Blogger](https://img.shields.io/badge/Blogger-FF5722?style=for-the-badge&logo=blogger&logoColor=white)
![Firefox](https://img.shields.io/badge/Firefox-FF7139?style=for-the-badge&logo=Firefox-Browser&logoColor=white)
![MySQL](https://img.shields.io/badge/mysql-4479A1.svg?style=for-the-badge&logo=mysql&logoColor=white)
![Reddit](https://img.shields.io/badge/Reddit-%23FF4500.svg?style=for-the-badge&logo=Reddit&logoColor=white)


# Credit to https://github.com/openscript/openscript :)


on:
  issue_comment:
    types: [created, edited, deleted]

name: Guestbook

jobs:
  update_guestbook:
    name: Update guestbook
    if: ${{ github.event.issue.title == 'Guestbook' }}
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v3
      - name: Update guestbook from comments
        uses: actions/github-script@v6
        with:
          script: |
            const truncateString = (str, num) => str.length > num ? str.slice(0, num) + "..." : str;

            const query = `query($owner:String!, $name:String!, $issue_number:Int!) {
              repository(owner:$owner, name:$name){
                issue(number:$issue_number) {
                  comments(first:5,orderBy:{direction:DESC, field:UPDATED_AT}) {
                    nodes {
                      author {
                        avatarUrl(size: 24)
                        login
                        url
                      }
                      bodyText
                      updatedAt
                    }
                  }
                }
              }
            }`;
            const variables = {
              owner: context.repo.owner,
              name: context.repo.repo,
              issue_number: context.issue.number
            }
            
            const result = await github.graphql(query, variables)
            const renderComments = (comments) => {
              return comments.reduce((prev, curr) => {
                let text = curr.bodyText.replace(/(\r\n|\r|\n)/g, "<br />").replace('|', '&#124;');
                text = truncateString(text, 125);
                return `${prev}| <a href="${curr.author.url}"><img width="24" src="${curr.author.avatarUrl}" alt="${curr.author.login}" /> ${curr.author.login}</a> |${new Date(curr.updatedAt).toLocaleString()}|${text}|\n`;
              }, "| Name | Date | Message |\n|---|---|---|\n");
            };
            const fileSystem = require('fs');
            const readme = fileSystem.readFileSync('README.md', 'utf8');
            fileSystem.writeFileSync('README.md', readme.replace(/(?<=<!-- Guestbook -->.*\n)[\S\s]*?(?=<!-- \/Guestbook -->|$(?![\n]))/gm, renderComments(result.repository.issue.comments.nodes)), 'utf8');
      - name: Push guestbook update
        run: |
          git config --global user.name 'Github Actions'
          git config --global user.email '41898282+github-actions[bot]@users.noreply.github.com'
          git commit -am ':sparkles: update guestbook'
          git push
