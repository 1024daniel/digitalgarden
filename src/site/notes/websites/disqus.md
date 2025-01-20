---
{"dg-publish":true,"permalink":"/websites/disqus/","noteIcon":"3"}
---


https://dg-docs.ole.dev/advanced/adding-custom-components/

https://www.reddit.com/r/ObsidianMD/comments/10pokli/i_built_a_digital_garden_using_obsidian_for_free/

https://help.disqus.com/en/articles/1717112-universal-embed-code

https://help.disqus.com/en/articles/1717111-what-s-a-shortname

#disqus

按照disqus和digital garden文档在`src/site/_includes/components/user/notes/footer`内嵌入文件
```html title:disqus.njk
<div id="disqus_thread"></div>
<script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT 
     *  THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR 
     *  PLATFORM OR CMS.
     *  
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: 
     *  https://disqus.com/admin/universalcode/#configuration-variables
     */
    /*
    var disqus_config = function () {
        // Replace PAGE_URL with your page's canonical URL variable
        this.page.url = PAGE_URL;  
        
        // Replace PAGE_IDENTIFIER with your page's unique identifier variable
        this.page.identifier = PAGE_IDENTIFIER; 
    };
    */
    
    (function() {  // REQUIRED CONFIGURATION VARIABLE: EDIT THE SHORTNAME BELOW
        var d = document, s = d.createElement('script');
        
        // IMPORTANT: Replace EXAMPLE with your forum shortname!
        s.src = 'https://iFlyWithWinds.disqus.com/embed.js';
        
        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>
    Please enable JavaScript to view the 
    <a href="https://disqus.com/?ref_noscript" rel="nofollow">
        comments powered by Disqus.
    </a>
</noscript>

```
这里的路径表示在非index其他note界面的footer添加disqus片段
同时为了不让digital garden的sidebar filetree遮挡住disqus片段，需要在`src/site/styles/custom-style.scss`添加对于disqus片段长度的格式代码
```css
#disqus_thread {
    width: 80%;
    max-width: 710px;
    margin: 3rem auto 0;
}

```