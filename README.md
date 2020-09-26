```
curl -s "https://qiita.com/api/v2/users/${user_id}/items?per_page=100" |jq -r ".[].url"|xargs -n1 -I{} bash -c "curl -s  {}.md > \$(echo {} | awk -F / '{print \$NF}').md"
curl -s "https://qiita.com/api/v2/users/${user_id}/items?per_page=100&page=2" |jq -r ".[].url"|xargs -n1 -I{} bash -c "curl -s  {}.md > \$(echo {} | awk -F / '{print \$NF}').md"
```
