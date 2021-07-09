# এখন আমরা জানবো কিভাবে প্লাগিন ডেভেলপ করতে হয়
## পার্ট-১ 😍
Basic requirments and Core Concept
* Action 
* Filter
* Shortcode
* widget

# আজকে আমরা জানবো কিভাবে ডাটা গুলি fetch করে WP List Table Class ব্যবহার করে pagination সহ ডাটাগুলিকে দেখতে পারি ❓ 
### ডাটাগুলিকে fetch করার জন্য আমরা একটি ফাঙ্কশন তৈরি করবো 
```
function wp_get_address( $arga=[]){
  global &wpdb
  
  $defaults = [
  "number"    => 20,
  "offset"    => 0,
  "orderby"   => id,
  "order"     => ASC
  ]
  
  $args = wp_parse_args($args, $defaults);
  
  // mysql query এখন আমরা fetch করবো 
  
  $results = $wpdb->get_results(
    $wpdb->prepare(" SELECT * FROM {$wpdb->prefix} ")
  );
  
  
  

}
```
#### defatuls যে চার টি প্যারামিটার ব্যবহার করতেছি তাদের অর্থ <br>
❤ number = fetch করে আমার কতটি object ডাটাবেস থেকে নিয়ে আসবো তার একটি নাম্বার দিবো ex(২০)<br>
❤ offset = pagination করার জন্য offset প্রয়োজন হয়। ex(০)<br>
❤ orderby ও order হচ্ছে self_expanotory<br>


