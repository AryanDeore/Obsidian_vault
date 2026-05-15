
There isn't a direct API or method provided by GitHub to crawl the GitHub Marketplace for all actions and their descriptions. However, you can manually search for actions and retrieve their details using the GitHub Marketplace interface or by programmatically accessing individual repositories hosting the actions.

If you're looking to programmatically gather information, here are some considerations:

1. **GitHub Marketplace Search**:
    
    - You can search for actions on the [GitHub Marketplace](https://github.com/marketplace/actions/) and manually browse through the listings to get the name and description.
2. **GitHub REST API**:
    
    - While the GitHub REST API doesn't provide a direct endpoint to list all Marketplace actions, you can use the API to search repositories tagged with `action.yml` or `action.yaml` files, which are metadata files required for GitHub Actions.
    - Example API endpoint: `GET /search/repositories?q=filename:action.yml`
    - This will return repositories containing an `action.yml` file, which you can then query to extract the action's metadata.
3. **Action Metadata**:
    
    - Each action's metadata file (`action.yml` or `action.yaml`) contains details like the name, description, and inputs. You can programmatically fetch this file from the repository to extract the information.
4. **Limitations**:
    
    - Crawling or scraping GitHub Marketplace directly is against GitHub's [Acceptable Use Policies](https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies). Ensure that your approach complies with these policies.

All the actions: https://github.com/marketplace?type=actions
Official actions: https://github.com/actions
