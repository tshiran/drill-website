The Apache Drill website is built using [Jekyll](http://jekyllrb.com/).

# Developing and Previewing the Website

To preview the website through GitHub Pages: [http://tshiran.github.io/drill-website](http://tshiran.github.io/drill-website)

To preview the website on your local machine:

```bash
jekyll serve --config _config.yml,_config-tlp.yml
```

To contribute changes to the site, submit a pull request.

# Compiling the Website

Once the website is ready, you'll need to compile the site to static HTML so that it can then be published to Apache. This is as simple as running the `jekyll build` command. The _config-incubator.yml and _config-tlp.yml configuration files make sure that the noindex meta tag is removed, and incorporate the correct base URL.

If the website will be hosted on Drill's Apache Incubator site (incubator.apache.org/drill):

```bash
jekyll build --config _config.yml,_config-incubator.yml
```

If the website will be hosted on Drill's Apache TLP site (drill.apache.org):

```bash
jekyll build --config _config.yml,_config-tlp.yml
```

# Uploading to the Apache Website (Drill Committers Only)

Apache project websites use a system called svnpubsub for publishing. Basically, the static HTML needs to be pushed by one of the committers into the Apache SVN.

```bash
svn co https://svn.apache.org/repos/asf/incubator/drill ../_site-apache
cp -R _site/* ../_site-apache/site/trunk/content/drill/
cd ../_site-apache/site/trunk/content/drill
```

Then `svn add` and `svn rm` as needed, and commit the changes via `svn commit -m "Website update"`.

Review the new website in the Apache staging area: [drill.staging.apache.org/drill](drill.staging.apache.org/drill)

Go to [https://cms.apache.org](https://cms.apache.org) (use your Apache credentials) and click on [Publish drill site](https://cms.apache.org/drill/publish).

