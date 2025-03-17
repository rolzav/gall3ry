# instructions

## current situation:
- main.css for styles;
- functions.php for init scripts;
- custom.js for js settings
- front-page.php for front end layout.

###

### main.css
```css
/* Masonry grid styles */
.masonry-grid {
  width: 100%;
  height: auto;
  display: grid;
  grid-template-columns: repeat(
    auto-fill,
    minmax(300px, 1fr)
  ); /* Flexible grid */
  gap: 20px; /* Add spacing between items */
}

/* Images and Masonry items */
.masonry-item {
  width: 100%;
  transition: transform 0.2s;
  cursor: pointer;
}

.masonry-item:hover {
  transform: translateY(-5px);
}

.img-fluid {
  width: 100%;
  height: auto;
  max-height: 300px; /* Controls the height of the images */
  object-fit: cover; /* Ensures the aspect ratio is maintained */
}

/* Responsive columns for different screen sizes */
@media (min-width: 576px) {
  .masonry-item {
    width: calc(50% - 10px); /* 2 columns on small screens */
  }
}

@media (min-width: 768px) {
  .masonry-item {
    width: calc(33.333% - 10px); /* 3 columns on medium screens */
  }
}

@media (min-width: 1200px) {
  .masonry-item {
    width: calc(25% - 10px); /* 4 columns on large screens */
  }
}

```

### functions.php
```php
<?php

/**
 * @package  Child theme custom
 *
 * @version 6.0.0
 */


// Exit if accessed directly
defined('ABSPATH') || exit;


/**
 * Enqueue scripts and styles
 */
add_action('wp_enqueue_scripts', 'bootscore_child_enqueue_styles');
function bootscore_child_enqueue_styles()
{

  // Compiled main.css
  $modified_bootscoreChildCss = date('YmdHi', filemtime(get_stylesheet_directory() . '/assets/css/main.css'));
  wp_enqueue_style('main', get_stylesheet_directory_uri() . '/assets/css/main.css', array('parent-style'), $modified_bootscoreChildCss);

  // style.css
  wp_enqueue_style('parent-style', get_template_directory_uri() . '/style.css');

  // custom.js
  // Get modification time. Enqueue file with modification date to prevent browser from loading cached scripts when file content changes. 
  $modificated_CustomJS = date('YmdHi', filemtime(get_stylesheet_directory() . '/assets/js/custom.js'));
  wp_enqueue_script('custom-js', get_stylesheet_directory_uri() . '/assets/js/custom.js', array('jquery'), $modificated_CustomJS, false, true);
}

/**
 * Enqueue Fancybox (version 3.5.7, stable) with a set of configuration options.
 */
function enqueue_fancybox()
{
  // Use Fancybox 3.5.7 from a popular CDN
  wp_enqueue_style(
    'fancybox-css',
    'https://cdnjs.cloudflare.com/ajax/libs/fancybox/3.5.7/jquery.fancybox.min.css',
    [],
    '3.5.7'
  );

  wp_enqueue_script(
    'fancybox-js',
    'https://cdnjs.cloudflare.com/ajax/libs/fancybox/3.5.7/jquery.fancybox.min.js',
    ['jquery'], // Depends on jQuery
    '3.5.7',
    true
  );

  // Masonry
  wp_enqueue_script('masonry-js', 'https://unpkg.com/masonry-layout@4.2.2/dist/masonry.pkgd.min.js', array('jquery'), '4.2.2', true);
}
add_action('wp_enqueue_scripts', 'enqueue_fancybox');



//
//
//

function get_images_from_directory($directory)
{
  $upload_dir = wp_upload_dir();
  $full_path = $upload_dir['basedir'] . '/' . $directory;

  if (!is_dir($full_path)) {
    return array();
  }

  $images = array();
  $files = scandir($full_path);

  foreach ($files as $file) {
    if ($file != '.' && $file != '..' && is_file($full_path . '/' . $file)) {
      $images[] = array(
        'url' => $upload_dir['baseurl'] . '/' . $directory . '/' . $file,
        'path' => $full_path . '/' . $file
      );
    }
  }

  return $images;
}

//
//
function initialize_fancybox()
{
  wp_enqueue_script('fancybox-js');
  wp_enqueue_style('fancybox-css');

  wp_add_inline_script('fancybox-js', '
      document.addEventListener("DOMContentLoaded", function() {
          $("[data-fancybox]").fancybox({
              protect: true,
              toolbar: {
                  show: true,
                  zoom: false,
                  share: false,
                  fullscreen: false,
                  download: false,
                  transition: false,
                  close: false
              },
              buttons: [
                  "zoom",
                  "share",
                  "slideShow",
                  "fullScreen",
                  "download",
                  "close"
              ],
              transitionEffect: "slide", // Smooth transition effect
              image: {
                  zoom: true, // Enable image zooming
                  zoomSpeed: 300 // Duration of the zoom animation
              },
              click: "close" // Close the lightbox on click
          });
      });
  ');
}
add_action('wp_enqueue_scripts', 'initialize_fancybox');

//
//
//
function enqueue_masonry()
{
  wp_enqueue_script(
    'masonry',
    'https://unpkg.com/masonry-layout@4.2.2/dist/masonry.pkgd.min.js',
    array('jquery'), // Masonry depends on jQuery
    '4.2.2',
    true
  );
}
add_action('wp_enqueue_scripts', 'enqueue_masonry');

//
//
function initialize_masonry()
{
  wp_add_inline_script('masonry', '
      document.addEventListener("DOMContentLoaded", function() {
          const grid = document.querySelector(".masonry-grid");
          if (grid) {
              new Masonry(grid, {
                  itemSelector: ".masonry-item",
                  columnWidth: ".masonry-item",
                  percentPosition: true,
                  gutter: 20 // Adjust the spacing between items
              });
          }
      });
  ');
}
add_action('wp_enqueue_scripts', 'initialize_masonry');
```


### custom.js
```javascript
document.addEventListener("DOMContentLoaded", function () {
  $("[data-fancybox]").fancybox({
    protect: true,
    toolbar: {
      show: true,
      zoom: false,
      share: false,
      fullscreen: false,
      download: false,
      transition: false,
      close: false,
    },
    buttons: ["zoom", "share", "slideShow", "fullScreen", "download", "close"],
  });
});




document.addEventListener("DOMContentLoaded", function () {
  const grid = document.querySelector(".masonry-grid");

  if (grid) {
    // Initialize Masonry
    new Masonry(grid, {
      itemSelector: ".masonry-item",
      columnWidth: ".masonry-item",
      percentPosition: true,
      gutter: 20 /* Adjust the spacing between items */,
      fitWidth: true,
    });
  }
});

```


### front-page.php
```php
<?php

/** * Front Page Template * * @package WordPress * @subpackage YourTheme */
get_header();
?>
<div id="primary" class="content-area">
    <main id="main" class="site-main">
        <div class="container">
            <div class="masonry-grid">
                <?php
                // Get images from the specified directory
                $images = get_images_from_directory('atteli/frontpageimages');
                if (!empty($images)) {
                    foreach ($images as $image) {
                ?>
                <div class="masonry-item col-12 col-sm-6 col-md-4 col-xl-3">
                    <a href="<?php echo $image['url']; ?>" class="fancybox" data-fancybox="gallery">
                        <img src="<?php echo $image['url']; ?>" alt="Gallery Image" class="img-fluid">
                    </a>
                </div>
                <?php
                    }
                } else {
                    ?>
                <div class="col-12 text-center py-5">
                    <p>No images found in the gallery.</p>
                </div>
                <?php
                }
                ?>
            </div>
        </div>
    </main>
</div>
<?php get_footer(); ?>
```

###

Please, let assist by re-factoring the whole files if needed, to initialize proper mansory gallery 

where I have specific folder created under UPLOADS folder..

I have following path for fetching images :

uploads/atteli/frontpageimages/*


How to display mansory correctly always fit full screen and are complete responsive mansory
