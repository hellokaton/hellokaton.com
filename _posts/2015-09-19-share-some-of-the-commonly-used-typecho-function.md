---
layout: post
title: 分享一些Typecho中常用的调用函数
categories: web
tags: typecho
---

## 下面是相关的函数

1、站点名称

```php
<?php $this->options->title() ?>
```

2、站点网址

```php
<?php $this->options ->siteUrl(); ?>
```

3、完整路径标题如分享几个Typecho中常用的调用函数

```php
<?php $this->archiveTitle(' &raquo; ', < span class="string">'', ' | '); ?><?php $this ->options->title(); ?>
```

<!-- more -->

4、站点说明

```php
<?php $this->options->description() ?>
```

5、模板文件夹地址

```php
<?php $this->options->themeUrl(); ?>
```

6、导入模板文件夹内的php文件

```php
<?php $this< /span>->need('.php'); ?> 
```

7、文章或者页面的作者

```php
<?php $this->author(); ?>
```

8、作者头像

```php
< ?php $this->author->gravatar('40') ?>
```

此处输出的完整的img标签，40是头像的宽和高。

9、该文作者全部文章列表链接

```php
<?php $this->author->permalink (); ?>
```

10、该文作者个人主页链接

```php
<?php $this->author->url(); ?>
```

11 、该文作者的邮箱地址

```php
<?php $this->author->mail(); ?>
```

12、上一篇与下一篇调用代码

```php
<?php $this->thePrev(); ?> <?php $this->theNext(); ?>
```

13、判断是否为首页，输出相关内容

```php
<?php if ($this->is('index')): ?>
//首页输出内容
<?php else: ?>
//不是首页输出内容
< span><?php endif; ?>
```

14、文章或页面，评论数目

```php
<?php $this->commentsNum('No Comments', '1 Comment' , '%d Comments'); ?>
```

14、文章或页面，评论数目

```php
<?php $this->commentsNum('No Comments', '1 Comment' , '%d Comments'); ?>
```

15、截取部份文章（首页每篇文章显示摘要），350是字数

```php
<?php $this->excerpt(350, '.. .'); ?>
```

16、调用自定义字段（官方文档坑爹，竟然没有，博主自己摸索出来的）

```php
<?php $this->fields->fieldName ?>
```

17、RSS地址

```php
<?php $this->options->feedUrl(); ?>
```

18、获取最新post

```php
<?php $this->widget('Widget_Contents_Post_Recent', 'pageSize=8&type=category')->parse('<li><a href="{permalink}">{title}</a></li>'); ?>
```

19、纯文字分类名称，不带链接

```php
<?php $this->category(',', false); ?>
```

20、获取文章分类列表

```php
<ul>
<?php $this->widget('Widget_Metas_Category_List')
                ->parse('<li><a href="{permalink}">{name}</a> ({count})</li>'); ?>
</ul>
```

21、获取某分类post

```php
<ul>
<?php 
$this->widget('Widget_Archive@indexyc', 'pageSize=8&type=category', 'mid=1')
->parse('<li><a href="{permalink}" title="{title}">{title}</a></li>'); ?>
</ul>
```

22、获取最新评论列表

```php
<ul>
    <?php $this->widget('Widget_Comments_Recent')->to($comments); ?>
    <?php while($comments->next()): ?>
        <li><a href="<?php $comments->permalink(); ?>"><?php $comments->author(false); ?></a>: <?php $comments->excerpt(50, '...'); ?></li>
    <?php endwhile; ?>
</ul>
```

23、首页获取 最新文章 代码限制条数

```php
<?php while ($this->next()): ?>
<?php if ($this->sequence <= 3): ?>
html
<?php endif; ?>
<?php endwhile; ?>
```

24、获取最新评论列表第二个版本，只显示访客评论不显示博主也就是作者或者说自己发的评论

```php
<?php $this->widget('Widget_Comments_Recent','ignoreAuthor=true')->to($comments); ?>
    <?php while($comments->next()): ?>
    <li><a href="<?php $comments->permalink(); ?>"><?php $comments->author(false); ?></a>: <?php $comments->excerpt(50, '...'); ?></li>
<?php endwhile; ?>
```

25、获取文章时间归档

```php
<ul>
    <?php $this->widget('Widget_Contents_Post_Date', 'type=month&format=F Y')
                ->parse('<li><a href="{permalink}">{date}</a></li>'); ?>
</ul>
```

26、获取标签集合，也就是标签云

```php
<?php $this->widget('Widget_Metas_Tag_Cloud', 'ignoreZeroCount=1&limit=28')->to($tags); ?>
<?php while($tags->next()): ?>
<a href="<?php $tags->permalink(); ?>" class="size-<?php $tags->split(5, 10, 20, 30); ?>"><?php $tags->name(); ?></a>
<?php endwhile; ?>
```

27、调用该文相关文章列表

```php
<?php $this->related(5)->to($relatedPosts); ?>
    <?php if ($relatedPosts->have()): ?>    //这句也可以写成 if (count($relatedPosts->stack))
    <?php while ($relatedPosts->next()): ?>
        <li><a href="<?php $relatedPosts->permalink(); ?>" title="<?php $relatedPosts->title(); ?>"><?php $relatedPosts->title(); ?></a></li>
    <?php endwhile; ?>
    <?php else : ?>
        <li>无相关文章</li>
    <?php endif; ?>
```

28、隐藏head区域的程序版本和模版名称

```php
<?php $this->header("generator=&template="); ?>
```

29、获取读者墙

```php
<?php
$period = time() - 999592000; // 時段: 30 天, 單位: 秒
$counts = Typecho_Db::get()->fetchAll(Typecho_Db::get()
->select('COUNT(author) AS cnt','author', 'url', 'mail')
->from('table.comments')
->where('created > ?', $period )
->where('status = ?', 'approved')
->where('type = ?', 'comment')
->where('authorId = ?', '0')
->group('author')
->order('cnt', Typecho_Db::SORT_DESC)
->limit(25)
);
$mostactive = '';
$avatar_path = 'http://www.gravatar.com/avatar/';
foreach ($counts as $count) {
  $avatar = $avatar_path . md5(strtolower($count['mail'])) . '.jpg';
  $c_url = $count['url']; if ( !$c_url ) $c_url = Helper::options()->siteUrl;
  $mostactive .= "<a href='" . $c_url . "' title='" . $count['author'] . " (参与" . $count['cnt'] . "次互动)' target='_blank'><img src='" . $avatar . "' alt='" . $count['author'] . "的头像' class='avatar' width='32' height='32' /></a>\n";
}
echo $mostactive; ?>
```

30、登陆与未登录用户展示不同内容

```php
<?php if($this->user->hasLogin()): ?>
登陆可见
<?php else: ?>
未登录和登陆均可见
<?php endif; ?>
```

31、导航页面列表调用隐藏特定的页面 这个演示隐藏了album和search两个页面

```php
<ul>
<li<?php if($this->is('index')): ?> class="current"<?php endif; ?>><a href="<?php $this->options->siteUrl(); ?>">主页</a></li>
<?php $this->widget('Widget_Contents_Page_List')->to($pages); ?>
    <?php while($pages->next()): ?>
    <?php if (($pages->slug != 'album') && ($pages->slug != 'search')): ?>
    <li<?php if($this->is('page', $pages->slug)): ?> class="current"<?php endif; ?>><a href="<?php $pages->permalink(); ?>" title="<?php $pages->title(); ?>"><?php $pages->title(); ?></a></li>
    <?php endif; ?>
    <?php endwhile; ?>
</ul>
```

参数说明
9.0版typecho支出在后台管理页面编辑时选择隐藏页面

32、Typecho归档页面(牧风提供)

```php
<?php $this->widget('Widget_Contents_Post_Recent', 'pageSize=10000')->to($archives);
    $year=0; $mon=0; $i=0; $j=0;
    $output = '<div id="archives">';
    while($archives->next()):
        $year_tmp = date('Y',$archives->created);
        $mon_tmp = date('m',$archives->created);
        $y=$year; $m=$mon;
        if ($mon != $mon_tmp && $mon > 0) $output .= '</ul></li>';
        if ($year != $year_tmp && $year > 0) $output .= '</ul>';
        if ($year != $year_tmp) {
            $year = $year_tmp;
            $output .= '<h3 class="al_year">'. $year .' 年</h3><ul class="al_mon_list">'; //输出年份
        }
        if ($mon != $mon_tmp) {
            $mon = $mon_tmp;
            $output .= '<li><span class="al_mon">'. $mon .' 月</span><ul class="al_post_list">'; //输出月份
        }
        $output .= '<li>'.date('d日: ',$archives->created).'<a href="'.$archives->permalink .'">'. $archives->title .'</a> <em>('. $archives->commentsNum.')</em></li>'; //输出文章日期和标题
    endwhile;
    $output .= '</ul></li></ul></div>';
    echo $output;
?>
```
