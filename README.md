# .github-workflows-issue-bot.ymlname: Issue Bot

on:
  issues:
    types: [opened, edited]

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - name: Label Issues
        uses: actions/github-script@v4
        with:
          script: |
            const labels = {
              'bug': ['bug', 'error'],
              'feature': ['new feature', 'enhancement'],
              'doc': ['documentation', 'docs']
            };
            const issueTitle = context.payload.issue.title.toLowerCase();
            const issueBody = context.payload.issue.body.toLowerCase();
            let labelApplied = false;

            for (const [label, keywords] of Object.entries(labels)) {
              if (keywords.some(keyword => issueTitle.includes(keyword) || issueBody.includes(keyword))) {
                await github.issues.addLabels({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  issue_number: context.payload.issue.number,
                  labels: [label]
                });
                labelApplied = true;
                break;
              }
            }

            if (!labelApplied) {
              await github.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.payload.issue.number,
                body: 'Por favor, proporciona más detalles, o consulta la documentación previa.'
              });
            }

            // Check for frequently asked questions and close the issue.
            const faqs = ['frequently asked question 1', 'frequently asked question 2'];
            if (faqs.some(faq => issueBody.includes(faq.toLowerCase()))) {
              await github.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.payload.issue.number,
                body: 'Gracias por tu consulta. Como es una duda frecuente, este issue será cerrado.'
              });
              await github.issues.update({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.payload.issue.number,
                state: 'closed'
              });
            }
