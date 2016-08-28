---
title: Et Hugo powered site
date: 2016-08-27
slug: et-hugo-powered-site
---

Tidligere på ugen deltog jeg i Praqmas [CoDe Academy](http://www.code-conf.com/academy2016/aar/), som handlede om continous delivery.
I den forbindelse fik jeg øjnene op for [GitLab](https://about.gitlab.com/), og særligt deres integrerede og gratis continous integration service.
En ting førte til en anden, og pludselig læste jeg om GitLab Pages, hvor man gratis kan hoste et statisk site med eget domæne, og ved hjælp af deres CI kunne sitet blive bygget hos dem.

## Hugo - Motoren bag
> A Fast and Flexible Static Site Generator - https://gohugo.io/

Ved hjælp af velstrukturerede markdown filer og templates genererer Hugo et statisk site. Filerne bliver gemt i mappen `./public`, og det er denne mappe man skal uploade til sin host. Det er så her, at det smarte kommer ind i billedet når man bruger Gitlab. Man skal nemlig bare pushe til sit repo og så genererer Gitlabs CI runner sitet ved hjælp af Hugo på deres servere.

## Gitlab + Hugo
Jeg har lavet et almindeligt og public [git repo](https://gitlab.com/henninghorn/henninghorn.dk) hos Gitlab, hvor jeg pusher min Hugo “kildekode”. Udover mine Hugo filer indeholder mit repo en `.gitlab-ci.yml` fil, som har følgende indhold:

```
image: alpine:3.4

before_script:
  - apk update && apk add openssl
  - wget https://github.com/spf13/hugo/releases/download/v0.16/hugo_0.16_linux-64bit.tgz
  - echo "37ee91ab3469afbf7602a091d466dfa5  hugo_0.16_linux-64bit.tgz" | md5sum -c
  - tar xf hugo_0.16_linux-64bit.tgz && cp ./hugo /usr/bin
  - hugo version

test:
  script:
  - hugo
  except:
  - master

pages:
  script:
  - hugo
  artifacts:
    paths:
    - public
  only:
  - master
```

I al sin enkelthed fortæller filen, at vi skal bruge Docker billedet “alpine” tagget “3.4”, og derefter instruerer vi CI runneren i hvordan den skal bygge vores projekt. Filen indeholder 2 steps/jobs, `test` og `pages`, men inden da kører den 5 kommandoer beskrevet i `before_script`.

I det sidste step beholder vi det genereret site ved at angive mappen (`public`) som et artifact. Hele processen fra man pusher til at sitet bliver deployet kan man følge via sit repos pipeline. Se [eksempel på pipeline](https://gitlab.com/henninghorn/henninghorn.dk/pipelines).

## Og hvad så
Godt spørgsmål! Personligt synes jeg, at det er et cool workflow, og en smart måde at holde hele sit site versionsstyret og hostet hos en 3. part, hvor man ikke er afhængig af database og et server side sprog.

Men det allermest spændende synes jeg er Gitlabs CI runner. I ovenstående eksempel bruges den bare til at generere et statisk site, men mulighederne er uendelige. Jeg eksempelvis haft succes med at lave en pipeline, som bestod af et test, hvor jeg kørte mine unit tests, og hvis de bestod og det samtidig var master branchen man arbejde med, så var deploy jobbet sat op til at deploye projektet ud på Heroku.

Alt sammen gratis! 🎉