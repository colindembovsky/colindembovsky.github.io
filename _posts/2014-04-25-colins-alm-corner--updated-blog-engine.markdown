---
layout: post
title: Colin’s ALM Corner – Updated Blog Engine
date: '2014-04-25 18:26:08'
tags:
- news
---

I have been using [Blogger](http://blogger.com) ever since I started my blog back in 2010. Once you get the template right (and set up a domain) it’s not a bad hosting platform. It works nicely with Windows Live Writer (as every self-respecting blog engine should). However, I felt it was time for a change – I wanted to take charge of my own blogging platform.

A couple of week’s ago I read a post by [Scott Hanselman](https://www.hanselman.com/) about [Mad’s Krisensen’s](http://madskristensen.net/) [MiniBlog](https://github.com/madskristensen/miniblog) engine. I had a look and liked it instantly – but there was no way to port from Blogger to MiniBlog. So I left it to stew in the back of my mind (\*ominous chuckle - BWAHAHAHA\*).

## Porting to MiniBlog from Blogger

I finally had another look a few days ago to see if I could port my existing blog posts over. While there was no native way to do this, I found a util ([BloggerBackup](http://bloggerbackup.codeplex.com/)) that let me export my blog posts (in ATOM format). I promptly exported all my posts.

The next trick was to import them into MiniBlog format. Fortunately there’s a little util that converts from BlogEngine.NET (or WordPress) to MiniBlog called [MiniBlogFormatter](https://github.com/madskristensen/MiniBlogFormatter). I cloned the repo and wrote my own formatter. This wasn’t too hard – using some [Linq-to-XML](http://msdn.microsoft.com/en-us/library/bb387061.aspx) I had something going pretty quickly. Here’s the code:

    public class BloggerATOMFormatter
    {
        public void Format(string originalFolderPath, string targetFolderPath)
        {
            FormatPosts(originalFolderPath, targetFolderPath);
        }
    
        private void FormatPosts(string originalFolderPath, string targetFolderPath)
        {
            var oldPostList = new Dictionary&lt;string, string&gt;();
            foreach (string file in Directory.GetFiles(originalFolderPath, "*.xml").Where(s =&gt; !s.EndsWith("comments.xml")))
            {
                var originalDoc = LoadDocument(file);
                XNamespace atomNS = @"http://www.w3.org/2005/Atom";
    
                var entry = originalDoc.Element(atomNS + "entry");
    
                var title = entry.Element(atomNS + "title").Value;
                var oldUrl = (from link in entry.Elements(atomNS + "link")
                              where link.Attributes().ToList().Any(a =&gt; a.Name == "rel" &amp;&amp; a.Value == "alternate")
                              select link).First().Attribute("href").Value.Replace("http://www.colinsalmcorner.com", "");
                
                var content = FixContent(entry.Element(atomNS + "content").Value);
                var publishDate = DateTime.Parse(entry.Element(atomNS + "published").Value);
                var lastModDate = DateTime.Parse(entry.Element(atomNS + "updated").Value);
                var slug = FormatterHelpers.FormatSlug(title);
                var categories = from cat in entry.Elements(atomNS + "category")
                                 select cat.Attribute("term").Value;
    
                var post = new Post();
                post.Author = "Colin Dembovsky";
                post.Categories = categories.ToArray();
                post.Content = content;
                post.IsPublished = true;
                post.PubDate = publishDate;
                post.Title = title;
                post.Slug = slug;
                post.LastModified = lastModDate;
                post.Comments = GetCommentsForPost(file);
    
                var newId = Guid.NewGuid().ToString();
                Storage.Save(post, Path.Combine(targetFolderPath, newId + ".xml"));
                oldPostList[oldUrl] = newId;
            }
            SaveOldPostMap(targetFolderPath, oldPostList);
        }
    
        private void SaveOldPostMap(string targetFolderPath, Dictionary&lt;string, string&gt; oldPostList)
        {
            var mapElement = new XElement("OldPostMap");
            foreach(var key in oldPostList.Keys)
            {
                mapElement.Add(
                    new XElement("OldPost",
                        new XAttribute("oldUrl", key),
                        new XAttribute("postId", oldPostList[key])
                    )
                );
            }
            var doc = new XDocument(mapElement);
            doc.Save(Path.Combine(targetFolderPath, "oldPosts.map"));
        }
    
        private List&lt;Comment&gt; GetCommentsForPost(string file)
        {
            var commentsFile = file.Replace(".xml", ".comments.xml");
            if (!File.Exists(commentsFile))
            {
                return new List&lt;Comment&gt;();  
            }
    
            var commentsDoc = LoadDocument(commentsFile);
            XNamespace atomNS = @"http://www.w3.org/2005/Atom";
    
            var list = new List&lt;Comment&gt;();
            foreach (var originalComment in commentsDoc.Descendants(atomNS + "entry"))
            {
                var authorElement = originalComment.Element(atomNS + "author");
                var name = authorElement.Element(atomNS + "name").Value;
                var email = authorElement.Element(atomNS + "email").Value;
                var uriElement = authorElement.Element(atomNS + "uri");
                string website = null;
                if (uriElement != null)
                {
                    website = uriElement.Value;
                }
    
                var content = originalComment.Element(atomNS + "content").Value;
                var publishDate = DateTime.Parse(originalComment.Element(atomNS + "published").Value);
    
                var comment = new Comment();
                comment.Author = name;
                comment.Email = email;
                comment.PubDate = publishDate;
                comment.Content = content;
                comment.IsAdmin = false;
                comment.Website = website;
                list.Add(comment);
            }
    
            return list.OrderBy(c =&gt; c.PubDate).ToList();
        }
    
        private string FixContent(string originalContent)
        {
            var regex = new Regex("&lt;pre class=\"brush: \\w*;\"&gt;(.*?)&lt;/pre&gt;", RegexOptions.IgnoreCase);
            foreach(Match match in regex.Matches(originalContent))
            {
                var formatted = match.Groups[1].Value.Replace("&lt;br /&gt;", Environment.NewLine);
                originalContent = originalContent.Replace(match.Groups[1].Value, formatted);
            }
            return originalContent.Replace("&lt;p&gt;&lt;/p&gt;&lt;br /&gt;", "").Replace("&lt;p&gt;&lt;/p&gt;", "").Replace("&lt;h3&gt;", "&lt;h2&gt;").Replace("&lt;/h3&gt;", "&lt;/h2&gt;");
        }
    
        private XDocument LoadDocument(string file)
        {
            return XDocument.Parse(File.ReadAllText(file));
        }
    }

There is a bit of “colinsALMcorner” specific code here, but if you’re looking to move from Blogger to MiniBlog you should be able to use most of this code. I had some issues with the formatting of the \<pre\> sections for [Syntax Highlighter](http://alexgorbatchev.com/SyntaxHighlighter/) – once I had that sorted, the formatter worked flawlessly.

## Redirecting Existing Posts

One of the challenges I had was what about search engines that already reference existing posts? Since I wanted to host MiniBlog on Azure and point my domain to the new site, I wanted to preserve any existing reference. However, the naming scheme for posts in Blogger is different from that in MiniBlog.

What I ended up doing was creating a map file as part of my convert-from-blogger-file-to-MiniBlog-file in the MiniBlogFormatter. I then created a simple HttpHandler that can server a “301 Moved Permanently” redirect when you hit an old post. Here’s the code:

    public class OldPostHandler : IHttpHandler
    {
        public bool IsReusable
        {
            get { return false; }
        }
    
        public void ProcessRequest(HttpContext context)
        {
            var oldUrl = context.Request.RawUrl;
            var oldPost = Storage.GetOldPost(oldUrl);
    
            if (oldPost == null)
            {
                throw new HttpException(404, "The post does not exist");
            }
    
            var newUrl = "/post/" + oldPost.Slug;
            context.Response.Status = "301 Moved Permanently";
            context.Response.AddHeader("Location", newUrl);
        }
    }

It’s small, neat and quick – keeping in line with the MiniBlog philosophy. Here’s the Storage.GetOldPost() method:

    public static Post GetOldPost(string url)
    {
        var map = GetOldPostMap();
        if (map.ContainsKey(url))
        {
            return GetAllPosts().SingleOrDefault(p =&gt; p.ID == map[url]);
        }
        return null;
    }
    
    public static Dictionary&lt;string, string&gt; GetOldPostMap()
    {
        GetAllPosts();
    
        if (HttpRuntime.Cache["oldPostMap"] != null)
        {
            return (Dictionary&lt;string, string&gt;)HttpRuntime.Cache["oldPostMap"];
        }
        return new Dictionary&lt;string, string&gt;();
    }
    
    private static void LoadOldPostMap()
    {
        var map = new Dictionary&lt;string, string&gt;();
        var mapFile = Path.Combine(_folder, "oldPosts.map");
        if (File.Exists(mapFile))
        {
            var doc = XDocument.Load(mapFile);
            foreach (var mapping in doc.Descendants("OldPost"))
            {
                var oldUrl = mapping.Attribute("oldUrl").Value;
                var newId = mapping.Attribute("postId").Value;
                map[oldUrl] = newId;
            }
        }
        HttpRuntime.Cache.Insert("oldPostMap", map);
    }

GetAllPosts() add’s a call to LoadOldPostMap() which finds the map file and reads it into memory. I only have 87 posts, so it’s not too heavy.

Here’s the code to invoke the handler in web.config:

    &lt;handlers&gt;
      &lt;remove name="CommentHandler"/&gt;
      &lt;add name="CommentHandler" verb="*" type="CommentHandler" path="/comment.ashx"/&gt;
      &lt;remove name="PostHandler"/&gt;
      &lt;add name="PostHandler" verb="POST" type="PostHandler" path="/post.ashx"/&gt;
      &lt;remove name="MetaWebLogHandler"/&gt;
      &lt;add name="MetaWebLogHandler" verb="POST,GET" type="MetaWeblogHandler" path="/metaweblog"/&gt;
      &lt;remove name="FeedHandler"/&gt;
      &lt;add name="FeedHandler" verb="GET" type="FeedHandler" path="/feed/*"/&gt;
      &lt;remove name="FeedsHandler"/&gt;
      &lt;add name="FeedsHandler" verb="GET" type="FeedHandler" path="/feeds/*"/&gt;
      &lt;remove name="CssHandler"/&gt;
      &lt;add name="CssHandler" verb="GET" type="MinifyHandler" path="*.css"/&gt;
      &lt;remove name="JsHandler"/&gt;
      &lt;add name="JsHandler" verb="GET" type="MinifyHandler" path="*.js"/&gt;
      &lt;remove name="OldPostHandler"/&gt;
      &lt;add name="OldPostHandler" verb="GET" type="OldPostHandler" path="*.html"/&gt;
    &lt;/handlers&gt;

You’ll see that I also added a “FeedsHandler” as well to work with the blogger feeds format, so that existing subscribers wouldn’t be affected by the switch (hopefully).

I then styled the site (since it’s based on bootstrap that wasn’t a problem). I also added a tag-cloud function and a search function. Both turned out to be really simple.

### Tag Cloud

I needed a method that would return all the categories and their frequency for the tag cloud. Here’s the code in the backend:

    public static Dictionary&lt;string, int&gt; GetTags()
    {
        var categories = Storage.GetAllPosts().SelectMany(p =&gt; p.Categories).Distinct();
        var tags = new Dictionary&lt;string, int&gt;();
        foreach(var cat in categories)
        {
            var count = Storage.GetAllPosts().Where(p =&gt; p.Categories.Any(c =&gt; c.Equals(cat, StringComparison.OrdinalIgnoreCase))).Count();
            tags[cat] = count;
        }
        return tags;
    }

Next I had to find a way to present a tag cloud on the page using javascript. There are lots of ways of doing this – I ended up using this [jQuery tagcloud script](https://github.com/addywaddy/jquery.tagcloud.js). Here’s the html for my tag cloud:

    &lt;div id="tagcloud"&gt;
        @{
            var tags = Blog.GetTags();
            foreach (var tag in tags.Keys)
            {
                &lt;a href="/category/@tag" rel="@tags[tag]"&gt;@tag&lt;/a&gt;
            }
        }
    &lt;/div&gt;
    
    &lt;script type="text/javascript"&gt;
        // tag cloud script
        $("#tagcloud a").tagcloud({
            size: {
                start: 0.8,
                end: 1.75,
                unit: 'em'
            },
            color: {
                start: "#7cc0f4",
                end: "#266ca2"
            }
        });
    &lt;/script&gt;

### Search

I regularly search my own blog – it’s a “working journal” of sorts. Having a search function was pretty important to me. Again the solution was really simple. Here’s the search code:

    public static List&lt;Post&gt; Search(string term)
    {
        term = term.ToLower();
        return (from p in Storage.GetAllPosts()
                where p.Title.ToLower().Contains(term) || p.Content.ToLower().Contains(term) || p.Comments.Any(c =&gt; c.Content.ToLower().Contains(term))
                select p).ToList();
    }

Once I had the results, I created a new search.cshtml page that shows just the first few lines of the blog post:

    @{
        var term = Request.QueryString["term"];
    
        Page.Title = Blog.Title;
        Layout = "~/themes/" + Blog.Theme + "/_Layout.cshtml";
        
        if (string.IsNullOrEmpty(term))
        {
            &lt;h1&gt;Oops!&lt;/h1&gt;
            &lt;p&gt;Something went wrong with your search. Try again...&lt;/p&gt;
        }
        else
        {
            &lt;h1&gt;Results for search: '@term'&lt;/h1&gt;
            
            var list = Blog.Search(term);
            if (list.Count == 0)
            {
                &lt;p&gt;No matches...&lt;/p&gt;
            }
            else
            {
                foreach(var p in list)
                {
                    @RenderPage("~/themes/" + Blog.Theme + "/PostSummary.cshtml", p);
                }
            }
        }
    }

The final bit was to get a search control. I ended up doing one entirely in css:

    input {
        outline: none;
    }
    input[type=search] {
        -webkit-appearance: textfield;
        -webkit-box-sizing: content-box;
        font-family: inherit;
        font-size: 80% !important;
    }
    input::-webkit-search-decoration,
    input::-webkit-search-cancel-button {
        display: none; /* remove the search and cancel icon */
    }
    
    /* search input field */
    input[type=search] {
        background: #ededed url(images/search-icon.png) no-repeat 9px center;
        border: solid 1px #ccc;
        padding: 5px 5px 5px 10px;
        width: 130px;
        
        -webkit-border-radius: 10em;
        -moz-border-radius: 10em;
        border-radius: 10em;
        
        -webkit-transition: all .5s;
        -moz-transition: all .5s;
        transition: all .5s;
    }
    input[type=search]:focus {
        width: 100%;
        background-color: #fff;
        border-color: #6dcff6;
        
        -webkit-box-shadow: 0 0 5px rgba(109,207,246,.5);
        -moz-box-shadow: 0 0 5px rgba(109,207,246,.5);
        box-shadow: 0 0 5px rgba(109,207,246,.5);
    }
    
    /* placeholder */
    input:-moz-placeholder {
        color: #999;
    }
    input::-webkit-input-placeholder {
        color: #999;
    }

And here’s the search control in my side-bar:

    &lt;section&gt;
        &lt;br /&gt;
        &lt;form action="/search" method="get" role="form" id="searchForm"&gt;
            &lt;fieldset&gt;
                &lt;input type="search" placeholder="Search this blog" name="term"&gt;
            &lt;/fieldset&gt;
        &lt;/form&gt;
        &lt;hr /&gt;
    &lt;/section&gt;

### Approve or Delete Comments from the Alert Mail

When someone writes a comment on a post, MiniBlog sends you an email. I like to moderate comments, so that’s how I’ve configured MiniBlog. In the mail there are 2 links – one to approve and one to delete the comment. However, I kept getting 403 “unauthorized” then clicking the links if I wasn’t logged in on the site. I made a small tweak to the CommentHandler Accept and Delete methods to redirect me to the login page instead of throwing a 403:

    if (!context.User.Identity.IsAuthenticated)
    {
        // was throwing 403 here
        FormsAuthentication.RedirectToLoginPage();
        return;
    }

Now when I hit the link from my mail, I get redirected to the login screen. Once logged in, the comment is approved/deleted and all’s well.

## Publishing to Azure

After testing posting from Windows Live Writer (no issues there) I then published the site to Azure. I changed my DNS records from Blogger to Azure and hey presto – new site is up!

## Conclusion

I’m really happy with the new look & feel and with the other modern web benefits (like SEO optimization and of course, speed) that MiniBlog brings. Thanks Mads!

I expect there may be a glitch or two for the switch over, but hopefully everything works well. Let me know in the comments if you experience any issues.

Happy reading!

