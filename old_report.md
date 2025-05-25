# security-incident-20240313-github-page
or _How to Takeover Thousands of Subdomains for Free Thanks to GitHub Pages_

Here's a brief analysis of a recent security incident I encountered regarding a subdomain takeover.

## Context
I own the domain `malvetro.corsica` (yes, `.corsica` TLD is a thing) on which a GitHub page is hosted at the root domain (`https://malvetro.corsica`). It's related to the venue my parents are renting ([malvetro GitHub repository](https://github.com/guisch/malvetro)). The website is created via [Publii](https://getpublii.com/). The domain registrar is [OVH](https://www.ovhcloud.com/fr/domains/).

While I was playing [Hell Divers 2](https://store.steampowered.com/app/553850/HELLDIVERS_2/), I received two emails from Google Search Console saying there is a new owner for `http://ftp.malvetro.corsica/`. This is weird as I do not recall creating that subdomain nor registering it on Google Search. 
The first email mentions `adetina180698@gmail.com` being the new owner of that subdomain, the second mentions `royricardo180798@gmail.com` being the owner of that same subdomain (both emails were sent at the same time).

![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/67cdc6d7-7135-4dac-a063-fdd445e66c04)
![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/0a04fcf6-863c-4cd7-a065-50cba1a1fbc2)


The first thing I do is to open `http://ftp.malvetro.corsica/` on the website. What a surprise!

![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/ac03ecca-8478-4c0e-b20f-6e83ce019230)

My first concern is that my OVH account has been compromised and that someone is changing the DNS records. I quickly check the website of the root domain and other subdomains, but **everything is OK**. Since I do not remember registering that domain, I open the OVH website to check the records.

![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/136a0a52-9ed7-4c6b-b0b6-760bd63f603d)

`ftp.malvetro.corsica` is indeed registered and points to the root domain with a CNAME record. I do not remember creating it, so my guess is that it was created by default by OVH like other CNAME records (like `autodiscover` and `autoconfig`). As mentioned earlier, the root domain `malvetro.corsica` is a GitHub page website, so that domain redirects to GitHub with A and AAAA records as mentioned in the GitHub [documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site).

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

A quick sanity check confirms that:
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

Since the compromised subdomain redirects with a CNAME to the root domain, which redirects to GitHub with A & AAAA for GitHub Pages, this means that the compromised subdomain is hosted on GitHub and is a GitHub Pages site.

To register a custom domain on GitHub Pages, you need to set up something on the repository which will create a file called `CNAME` ([documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)).

> Under "Custom domain", type your custom domain, then click Save. If you are publishing your site from a branch, this will create a commit that adds a CNAME file directly to the root of your source branch. If you are publishing your site with a custom GitHub Actions workflow, no CNAME file is created. For more information about your publishing source, see "Configuring a publishing source for your GitHub Pages site."

So the first thing I try is to search on GitHub for `ftp.malvetro.corsica`. No luck. I tried with variations and regex matching, but still no results.

![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/864ff1bd-753d-4982-b27b-b59de8417f85)

I see three reasons why there are no results:
1. The repository is too recent and not yet indexed by GitHub.
2. The repository is private (although, I'm not sure it's possible to have GitHub Pages with a private repo).
3. The repository is deleted.

I'll assume I can't find anything about my compromise because of reason 1. However, I can try to search if others have been compromised the same way. I go back to my compromised subdomain and copy the first word, `HANTUSLOT`, and search for that on GitHub.
![image](https://github.com/Guisch/security-incident-20240313-github-page/assets/1493619/2b00b7f0-d6c0-423a-ba69-f0dd13c97d1c)

Bingo, there are 4 results, with similar page titles. All results are GitHub Pages from 3 different GitHub accounts. 3 out of 4 are using the same `ftp.` subdomain. Digging into the accounts, one of them had 101 repositories of GitHub Pages with compromised subdomains.

You can find the list of all repos archived by user here: https://github.com/Guisch/security-incident-20240313-github-page/tree/main/repositories

Because I'm still frustrated not to be able to find the repo that actually used my subdomain, I use the gh CLI to scrape all recent repos with `ftp.` in the name, get all repo owners, and download all their repositories:
```bash
$ cd repositories
$ # Search for the last 1000 updated repos, if there is "ftp." in the repos name, then save the name of the repo owner to "accounts" file if not already present.
$ (gh search repos ftp --sort updated --limit 1000 | while read -r repo _; do if [ $(echo "$repo" | grep -E "\/ftp\.") ]; then echo $repo | cut -d/ -f1; fi; done) | sort -u > accounts
$ # For each repo owner in the "accounts" file, list at most 1000 of its repos and clone them into <owner>/<repo> directory.
$ while read -r user; do gh repo list "$user" --limit 1000 | while read -r repo _; do mkdir -p "$user"; git clone "https://github.com/$repo" "$repo"; done; done < accounts
$ # List all file names "CNAME" and check if one contains "malvetro"
$ find . -name 'CNAME' -exec grep -H "malvetro" {} \;
$
```

No results. A few days later, searching on GitHub still doesn't return anything, so my attacker must have removed the repo. Just in case, I adapted the previous script to also download git refs to be able to search in history. But still no luck.

To check if there are other common subdomains than `ftp` that are used, I run a quick frequency analysis on all the repos I cloned:
```bash
$ cd repositories
$ # For each repo owner in the "accounts" file, list at most 1000 of its repos and mirror clone them (to get history) in "<owner>/<repo>.mirror" directory.
$ while read -r user; do gh repo list "$user" --limit 1000 | while read -r repo _; do mkdir -p "$user"; GIT_TERMINAL_PROMPT=0 git clone --mirror "https://github.com/$repo" "$repo.mirror"; done; done < accounts
$ # For each cloned repository, cd into the directory then search for all additions (lines starting with a +) of a domain (starting with [a-zA-Z0-9]) in the "CNAME" file, save the addition to the file "subdomains" without the leading "+"
$ (TMP=$(pwd); for f in $(find $(pwd) -maxdepth 2 -mindepth 2 -type d); do cd $f; git log -p --follow --all -- CNAME | grep -E "^\+[a-zA-Z0-9]" | sed 's/^+//'; done; cd $TMP;) | sort -u > subdomains
$ # List all subdomain frequencies
$ cat subdomains | cut -d. -f1 | sort | uniq -c | sort -n
[...]
      5 status
      6 blog
      6 depo10bonus10
      6 depo20bonus20
      6 linkserverinternasional
      6 linkservervietnam
      7 beta
      7 docs
      7 linkserverasia
      7 linkserversensasional
      8 cpanel
      8 test
     13 duit188
     13 whm
     26 webmail
     37 www
     48 mail
   2091 ftp
```

You can find the list of all subdomains here: https://github.com/Guisch/security-incident-20240313-github-page/blob/main/repositories/subdomains.
`ftp` subdomains like mine were used **2091** times! There are likely subdomains in the list that are legitimate, but I don't see cases where one would host a GitHub Pages on an `ftp` subdomain. This confirms my belief that:
- Attackers are using automated tools.
- `ftp` is generated by default on some registrars.

The subdomain takeover is quite well-known. Take, for example, [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz), a superb tool for checking how to takeover domains or dangling subdomains with unused CNAME DNS records like mine. This is actually discussed in this issue: https://github.com/EdOverflow/can-i-take-over-xyz/issues/68#issuecomment-1949450029. GitHub actually "warns" about it in the documentation:
> We recommend verifying your custom domain prior to adding it to your repository, in order to improve security and avoid takeover attacks.

However, here the source of the issue is not on a legacy subdomain that wasn't properly decommissioned but a subdomain generated by default by the registrar.

## Hypothesis

My guess on how attackers have automated their thousands takeovers:
- List all domains that point to [GitHub Pages IPs](https://api.github.com/meta).
- Dig for subdomains with CNAME redirection to the source domain (subdomain created by default by registrar or leftovers like `ftp`, `mail`, or any [common subdomain](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt)).
- Create a GitHub repo, enable GitHub Pages for the found subdomain.
- Voilà, you have a subdomain takeover hosted by GitHub.

## Where I Was Lucky

- Having set up Google Search Console with domain verification done via a DNS record informed me about anyone that wants to claim the Search Console ownership of any subdomain.
- I work in cybersecurity which allowed me to quickly understand the schema. I believe the compromise path isn't trivial and requires some technical knowledge to identify (less tech-savvy people could just think they have been hacked).

## Remediations

- I quickly removed the `ftp` CNAME record on my registrar right after receiving the email. _That's probably why I wasn't able to track down my attacker in the end._
- [Verify custom domain for GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages)

## Recommendations

### For GitHub
- Assume I have a GitHub Page repo with a custom domain setup but not verified, GitHub should send me an email if someone else is using a subdomain for a GitHub page to alert me. The email should mention the doc to verify the custom domain and help to remediate the issue.
- To remediate the current issue, look for all GitHub pages repo, check the root domain, check if that root domain is used for a GitHub Page by another user. From there, you have a list of suspicious domains.

### For OVH
- Do not create DNS records by default without warning the user.

### For me
- Control & monitor my DNS records
