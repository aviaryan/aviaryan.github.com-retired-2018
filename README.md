# My Site (Retired)

> After using Jekyll for 5 years, I am now closing this site in favor of new technologies like Gatsby. The new website is at [aviaryan/aviaryan.com](https://github.com/aviaryan/aviaryan.com). Cheers! - 2/12/2018

Built with Jekyll.  

<s>Visit [https://aviaryan.in](https://aviaryan.in)</s>

Visit [https://aviaryan.com](https://aviaryan.com)
  

```
Copyright 2013-18 Avi Aryan  

Licensed under the Apache License, Version 2.0 (the "License");  
you may not use this file except in compliance with the License.  
You may obtain a copy of the License at  

http://www.apache.org/licenses/LICENSE-2.0  

Unless required by applicable law or agreed to in writing, software  
distributed under the License is distributed on an "AS IS" BASIS,  
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
See the License for the specific language governing permissions and  
limitations under the License.  
```


### Running

Using Docker

```sh
docker run -t --rm -v "$PWD":/usr/src/app -v site:/usr/src/app/_site -p "4000:4000" --env JEKYLL_ENV=local starefossen/github-pages
# kill later
docker kill $(docker ps -q)
```

Thanks to https://github.com/Starefossen/docker-github-pages.


#### Production-local handling - 

```
{% if jekyll.environment == "local" %}
local
{% else %}
prod
{% endif %}
```
