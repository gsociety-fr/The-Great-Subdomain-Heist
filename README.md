# security-incident-20240313-github-page
A brief analysis about a recent security incident I encountered about a subdomain takeover

# Context
I own the domain `malvetro.corsica` (yes, `.corsica` TLD is a thing) on which a github page is hosted at the root domain (`https://malvetro.corsica`). _It's about the venue my parents are renting (https://github.com/guisch/malvetro)_. The website is created via [Publii](https://getpublii.com/).
The domain registraar is [OVH](https://www.ovhcloud.com/fr/domains/). 

While I was playing [Hell Divers 2](https://store.steampowered.com/app/553850/HELLDIVERS_2/), I received 2 emails from Google Search Console saying there is a new owner for `http://ftp.malvetro.corsica/`. This is weird as I do not recall creating that subdomain nor registering it on Google Search.
The first email mention `adetina180698@gmail.com` being the new owner of that subdomain, the second mention `royricardo180798@gmail.com` being the owner of that same subdomain (both email were sent at the same time).
![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/67cdc6d7-7135-4dac-a063-fdd445e66c04)
![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/0a04fcf6-863c-4cd7-a065-50cba1a1fbc2)


The first thing I do is to open `http://ftp.malvetro.corsica/` on the website. What a surprise !
![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/ac03ecca-8478-4c0e-b20f-6e83ce019230)

My first concerns is that my OVH account has been compromised and that someone is changing the DNS records. I'm quickly checking the website of the root domain and other subdomains, but **everything is OK**.
Since I do not remember registering that domain, I open OVH website to check the records.
![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/136a0a52-9ed7-4c6b-b0b6-760bd63f603d)

`ftp.malvetro.corsica` is indeed registered and point to the root domain with a CNAME record. I do not remember creating it so my guess is that it was created by default by OVH like other CNAME records (like `autodiscover` and `autoconfig`).
As mentioned earlier, the root domain `malvetro.corsica` is a github page website, so that domain redirect to Github with A and AAAA records as mentioned in the Github [documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site).
> Navigate to your DNS provider and create either an ALIAS, ANAME, or A record. You can also create AAAA records for IPv6 support. If you're implementing IPv6 support, we highly recommend using an A record in addition to your AAAA record, due to slow adoption of IPv6 globally. For more information about how to create the correct record, see your DNS provider's documentation.
> - To create A records, point your apex domain to the IP addresses for GitHub Pages.
> ```
> 185.199.108.153
> 185.199.109.153
> 185.199.110.153
> 185.199.111.153
> ```
> - To create AAAA records, point your apex domain to the IP addresses for GitHub Pages.
> ```
> 2606:50c0:8000::153
> 606:50c0:8001::153
> 2606:50c0:8002::153
> 2606:50c0:8003::153
> ```

A quick sanity check confirm that:
```
$ dig malvetro.corsica +noall +answer -t A
malvetro.corsica.       442     IN      A       185.199.110.153
malvetro.corsica.       442     IN      A       185.199.109.153
malvetro.corsica.       442     IN      A       185.199.108.153
malvetro.corsica.       442     IN      A       185.199.111.153
$ dig malvetro.corsica +noall +answer -t AAAA
malvetro.corsica.       461     IN      AAAA    2606:50c0:8002::153
malvetro.corsica.       461     IN      AAAA    2606:50c0:8003::153
malvetro.corsica.       461     IN      AAAA    2606:50c0:8001::153
malvetro.corsica.       461     IN      AAAA    2606:50c0:8000::153
$ dig ftp.malvetro.corsica +noall +answer -t A
ftp.malvetro.corsica.   430     IN      CNAME   malvetro.corsica.
malvetro.corsica.       430     IN      A       185.199.108.153
malvetro.corsica.       430     IN      A       185.199.110.153
malvetro.corsica.       430     IN      A       185.199.109.153
malvetro.corsica.       430     IN      A       185.199.111.153
$ dig ftp.malvetro.corsica +noall +answer -t AAAA
ftp.malvetro.corsica.   421     IN      CNAME   malvetro.corsica.
malvetro.corsica.       444     IN      AAAA    2606:50c0:8001::153
malvetro.corsica.       444     IN      AAAA    2606:50c0:8002::153
malvetro.corsica.       444     IN      AAAA    2606:50c0:8000::153
malvetro.corsica.       444     IN      AAAA    2606:50c0:8003::153
```

Since the compromised subdomain redirect with a CNAME to the root domain, which redirect to Github with A & AAAA for Github Pages, this means that the compromised subdomain is hosted on Github and is a Github Pages.

To register a custom domain on Github Pages, you need to setup something on the repository which will create a file called `CNAME` ([doc](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain))
> Under "Custom domain", type your custom domain, then click Save. If you are publishing your site from a branch, this will create a commit that adds a CNAME file directly to the root of your source branch. If you are publishing your site with a custom GitHub Actions workflow, no CNAME file is created. For more information about your publishing source, see "Configuring a publishing source for your GitHub Pages site."

So the first thing I try is to search on Github for `ftp.malvetro.corsica`. No luck. Tried with variation and regex matching, still no results.
![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/864ff1bd-753d-4982-b27b-b59de8417f85)

I see three reasons why there's no results:
1. Repo is to recent and not yet indexed by Github
2. Repo is private (although, I'm not sure it's possible to have github pages with private repo)
3. Repo is deleted

I'll assume I can't find anything about my compromission because of reason 1. However, I can try to search if other has been compromised the same way ?
I go back on my compromised subdomain and copy the first word `HANTUSLOT` and search for that on Github
![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/2b00b7f0-d6c0-423a-ba69-f0dd13c97d1c)

Bingo, there is 4 results, with similar page titles. All results are Github Pages from 3 differents Github accounts. 3 of 4 are having the same `ftp.` subdomain. Digging in accounts, one of them was having 101 repositories of Github Pages with comprimised subdomain.
Here is a table that list all of them and the link to download 

| Username   | Repo                                               | Backup |
|------------|----------------------------------------------------|--------|
| Lomansanih | https://github.com/Lomansanih/Lomansanih.github.io |        |
| yomunsa    | https://github.com/yomunsa/yomunsa.github.io       |        |
| Ronie01    | https://github.com/Ronie01?tab=repositories        |        |

Because I was still frustrated to not be able to find the repo that actually used my subdomain, I used gh cli to scrape all repo with `ftp.` in the name, get all repo owners and download all their repositories:
```bash
$ cd repositories
$ # Search for the last 1000 updated repos, if there is "ftp." the the repos name, then save the name of the repo owner to "accounts" file if not already present.
$ (gh search repos ftp --sort updated --limit 1000 | while read -r repo _; do if [ $(echo "$repo" | grep -E "\/ftp\.") ]; then echo $repo | cut -d/ -f1; fi; done) | sort -u > accounts
$ # For each repo owner in the "accounts" file, list at most 1000 of its repo and clone them in <owner>/<repo> directory.
$ while read -r user; do gh repo list "$user" --limit 1000 | while read -r repo _; do mkdir -p "$user"; git clone "https://github.com/$repo" "$repo"; done; done < accounts
$ # List all file name CNAME and check if one contains "malvetro"
$ find . -name 'CNAME' -exec grep -H "malvetro" {} \;
$
```

Still no luck. It's now 2AM and I'm tired, time to go to bed and check again tomorrow if Github indexed anything.

# Remediation

- I'm just gonna remove the `ftp` record on my registraar
- Verify custom domain for Github Pages (https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages)
