---
title: Wordpress troubleshooting
date: 2025-03-21 15:23:44
tags:
---

在04服务器上用docker运行了一个wordpress,用来备份以前的blog。
想加一个页面显示所有帖子列表，于是折腾出一堆事情。
记录一下都遇到了哪些问题。

## 更新遇到mysql db版本问题

有文章说可以在wordpress的仪表盘->外观->编辑功能添加一个模板。但是我的外观下没有编辑功能，只有“主题文件编辑器”。里面可以编辑现有模板，不能添加新的。

于是以为是版本太低，于是运行docker-compose pull命令拉取最新版本。

更新后启动，显示数据库连不上。

用以下命令看log

```console
$ docker ps
CONTAINER ID   IMAGE                                  COMMAND                  CREATED       STATUS                 PORTS                                                                                                                                                                                    NAMES
f1e778094655   mysql:8.4.4                            "docker-entrypoint.s…"   8 hours ago   Up 8 hours             3306/tcp, 33060/tcp                                                                                                                                                                      wordpress_db_1
3b1b511ddee6   wordpress                              "docker-entrypoint.s…"   8 hours ago   Up 8 hours             0.0.0.0:4880->80/tcp, :::4880->80/tcp                                                                                                                                                    wordpress_wordpress_1
...

docker logs f1e7
...
```

上面命令可以显示log,log里面显示如下关键信息

```log
Invalid MySQL server upgrade: Cannot upgrade from 80033 to 90200 
```

mysql大版本升级不能直接升。于是上hub.docker.com里搜mysql,找到8.x版本最新版tag是8.4.4。

修改docker-compose.yml如下，加上版本号，恢复老的8.4.4 mysql

```yml
version: '3.1'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 4880:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html

  db:
    image: mysql:8.4.4
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
```

## 在volume里添加php文件

需要在wordpress里添加一个文件template-article-list.php，内容如下

```php
<?php
/*
Template Name: 文章列表带分页
*/
get_header(); 

// 获取当前页码
$paged = (get_query_var('paged')) ? get_query_var('paged') : 1;

// 创建自定义查询
$args = array(
    'post_type'      => 'post',
    'posts_per_page' => 50,
    'paged'          => $paged,
    'post_status'    => 'publish'
);

$custom_query = new WP_Query($args);
?>

<div class="article-list-container">
    <?php if ($custom_query->have_posts()) : ?>
        <ul class="article-list">
            <?php while ($custom_query->have_posts()) : $custom_query->the_post(); ?>
                <li class="article-item">
                    <span class="article-date"><?php echo get_the_date('Y-m-d'); ?></span>
                    <a class="article-title" href="<?php the_permalink(); ?>"><?php the_title(); ?></a>
                </li>
            <?php endwhile; ?>
        </ul>

        <!-- 分页导航 -->
        <div class="pagination">
            <?php
            echo paginate_links(array(
                'base'      => str_replace(999999999, '%#%', esc_url(get_pagenum_link(999999999))),
                'format'    => '?paged=%#%',
                'current'   => max(1, $paged),
                'total'     => $custom_query->max_num_pages,
                'prev_text' => __('« 上一页'),
                'next_text' => __('下一页 »'),
                'type'      => 'list'
            ));
            ?>
        </div>

    <?php else : ?>
        <p>没有找到文章</p>
    <?php endif; ?>

    <?php wp_reset_postdata(); ?>
</div>

<?php get_footer(); ?>
```

另外需要在css里添加如下内容

```css
.article-list-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
}

.article-item {
    margin-bottom: 15px;
    padding: 10px;
    border-bottom: 1px solid #eee;
}

.article-date {
    display: inline-block;
    width: 120px;
    color: #666;
}

.article-title {
    text-decoration: none;
    color: #333;
}

.pagination {
    margin-top: 30px;
    text-align: center;
}

.pagination ul {
    list-style: none;
    margin: 0;
    padding: 0;
}

.pagination li {
    display: inline-block;
    margin: 0 2px;
}
```

用如下命令操作docker volume里的文件

```console
$ docker volume ls
DRIVER    VOLUME NAME
local     wordpress_db
local     wordpress_wordpress
```

在没有vi命令的情况下靠echo创建文件

```console
$ docker exec -it 3b1b /bin/bash
root@3b1b511ddee6:/var/www/html# cd wp-content/themes/twentyfifteen
root@3b1b511ddee6:/var/www/html/wp-content/themes/twentyfifteen# pwd
/var/www/html/wp-content/themes/twentyfifteen
root@3b1b511ddee6:/var/www/html/wp-content/themes/twentyfifteen# echo <<EOF > template-article-list.php
<?php
/*
Template Name: 文章列表带分页
*/
?>
EOF
root@3b1b511ddee6:/var/www/html/wp-content/themes/twentyfifteen# chown www-data:www-data template-article-list.php
```

文件创建成功后就可以在wordpress的仪表盘->外观->主题文件编辑器 中看到这个文件，把完整代码粘贴进去。

css代码则附加在style.css最后。

准备好之后，新建页面，编辑，选择模板“文章列表带分页”。
