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

/**
 * Get the Product
 *
 * @return Array
 */

function lmfwppt_get_licenses( $args = [] ) {
    global $wpdb;

    $defaults = [
        'number'  => 20,
        'offset'  => 0,
        'orderby' => 'id',
        'order'   => 'ASC'
    ];

    $args = wp_parse_args( $args, $defaults );

    $last_changed = wp_cache_get_last_changed( 'license' );
    $key          = md5( serialize( array_diff_assoc( $args, $defaults ) ) );
    $cache_key    = "all:$key:$last_changed";

    $sql = $wpdb->prepare(
            "SELECT * FROM {$wpdb->prefix}license_manager
            ORDER BY {$args['orderby']} {$args['order']}
            LIMIT %d, %d",
            $args['offset'], $args['number']
    );

    $items = wp_cache_get( $cache_key, 'license' );

    if ( false === $items ) {
        $items = $wpdb->get_results( $sql );

        wp_cache_set( $cache_key, $items, 'license' );
    }

    return $items;
}

```


#### defatuls যে চার টি প্যারামিটার ব্যবহার করতেছি তাদের অর্থ <br>
❤ number = fetch করে আমার কতটি object ডাটাবেস থেকে নিয়ে আসবো তার একটি নাম্বার দিবো ex(২০)<br>
❤ offset = pagination করার জন্য offset প্রয়োজন হয়। ex(০)<br>
❤ orderby ও order হচ্ছে self_expanotory<br>

# এখন আমরা জানবো কিভাবে count করতে হয় 

### Count করার জন্য আমি ফাঙ্কশন তৈরি করবো 
```

/**
 * Get the Product Count
 *
 * @return int
 */


function product_count(){
  global $wpdb;
  return (int) $wpdb->get_var("SELECT count(id) FROM {$wpdb->prefix}tables_name");
}

```
## আমাদের function.php কাজ শেষ বা <br>

# এখন আমি জানবো কিভাবে WP list-table বানাতে হয় ❓
### সেই জন্য আমরা admin ফোল্ডার ভিতরে একটা ফাইল নিবো <br>
✔ admin/ProductsListTable.php নামে <br>
এর কাজ হবে list-table genarate করে <br>

### wp_list_table সরাসরি ব্যবহার করে যাই না। আমাদের কে extends করে কাজ করতে হবে। 

```
<?php 



if( ! class_exists('WP_List_Table')){
	require_once ABSPATH . 'wp-admin/includes/class-wp-list-table.php';
}


/**
 *  
 * Product List Table Class
 * 
*/


class Product_List extends \WP_List_Table(){
  function __construct(){
		এখন আমাদেরকে __construct() parent কে কল করতে হবে 
    এইখানে মাত্র কয়েকটি প্যারামিটার আছে। 
    'ajax'     => false করার কারণ হচ্ছে bulk গুলা কে এডিট করবো না।  
     parent::__construct([
			'singular' => "",
			'plural'   => "",
			'ajax'     => false
		]);
	}
  
  আমাদের কয় টি কলাম দরকার সেই আরো একটি ফাঙ্কশন use করতে হবে 
  public function get_columns(){
		return [
			'cb'     => "<input type='checkbox'/>",
			'name'   => __('Name','Name'),
			'slug'   => __('Slug','Slug'),
			'dated'  => __('Date','Date')
		];
	}
  
  protected function column_default($item, $column_name){
		switch ($column_name) {
			case 'value':
				# code...
				break;
			
			default:
				return isset($item->$column_name) ? $item->$column_name : '';
		}
	}
  
  ডিফল্ট যে কলাম গুলি আছে এ গুলিকে আমাকে কাস্টোমাইজ করতে হবে 
  সেই জন্য নিচের কোড গুলি দেখতে হবে 
  // Default column Customize
	public function column_name($item){
		$actions = [];
		$actions['edit']   = sprintf( '<a href="%s" title="%s">%s</a>', admin_url( 'admin.php?page=license-manager-wppt&action=edit&id=' . $item->id ), $item->id, __( 'Edit', 'license-manager-wppt' ), __( 'Edit', 'license-manager-wppt' ) );

        $actions['delete'] = sprintf( '<a href="%s" class="submitdelete" onclick="return confirm(\'Are you sure?\');" title="%s">%s</a>', wp_nonce_url( admin_url( 'admin-post.php?action=wd-ac-delete-address&id=' . $item->id ), 'wd-ac-delete-address' ), $item->id, __( 'Delete', 'license-manager-wppt' ), __( 'Delete', 'license-manager-wppt' ) );

		return sprintf(
			'<a href="%1$s"><strong>%2$s</strong></a> %3$s', admin_url('admin.php?page=license-manager-wppt&action=view&id' .$item->id), $item->name, $this->row_actions($actions)
		);
	}

  এখন আমরা জানবো কিভাবে pagination করতে হয় 
  // pagination and sortable use this code
    function get_sortable_columns() {
        $sortable_columns = [
            'name'       => [ 'name', true ],
            'dated' => [ 'dated', true ],
        ];

        return $sortable_columns;
    }
    // pagination and sortable use this code
    most important function টি হচ্ছে prepare_items
    public function prepare_items(){

		$column = $this->get_columns();
		$hidden = [];
		$sortable = $this->get_sortable_columns();

		$this->_column_headers = [$column, $hidden, $sortable];

		//  pagination and sortable
		$per_page     = 2;
        $current_page = $this->get_pagenum();
        $offset       = ( $current_page - 1 ) * $per_page;
        $args = [
            'number' => $per_page,
            'offset' => $offset,
        ];

        if ( isset( $_REQUEST['orderby'] ) && isset( $_REQUEST['order'] ) ) {
            $args['orderby'] = $_REQUEST['orderby'];
            $args['order']   = $_REQUEST['order'] ;
        }

        $this->items = wp_product($args);

        // pagination and sortable

		$this->set_pagination_args([
			'total_items' =>product_count(),
            'per_page'    =>$per_page,
		]);
	}

```
### এখন যদি আমরা Product_List ক্লাস তাকে instance ব্যবহার করতে চাই তাহলে 
 ✔ admin/templates/products/list.php ফাইল নিবো 
 যে ক্লাস টাকে আমরা ডিক্লেয়ার করেছিলাম সেই টাকে ইনহেরিট করবো। 
 ### আমাদের টেবিল টা হবে একটা form মধ্যে সেই জন্য নিচের কোড গুলি করতে হবে 
 ```
 <form action="" method="post">
    	<?php 
    		$table = new \admin\ProductsListTable();
    		$table->prepare_items();
    		$table->display();
    	?>
    </form>
 
 ```
 
 
 # আজকে আমরা জানবো কিভাবে ওয়ার্ডপ্রেস Ajax ব্যবহার করতে হয় 
 #### আমাদের ফর্ম যে ডাটাগুলি আছে ওই দাঁতগুলি কে একটি ভ্যারিয়েবল নিবো 
  ### self invoking function anonymous function ❓❓
  * এই ফাঙ্কশন টি নিজে নিজে কে কল করে 
  - self invoking function anonymous function তৈরি করে হয় যেন এই ফাঙ্কশন টি অন্য কোন ফাঙ্কশন সাথে মারামারি না করে। <br>
  
  
  ```
 ;(function($){
      var data = $(this).serialize(); // form ডাটা গুলি সব data ভ্যারিয়েবল রাখলাম 
 })(JQUERY)
 ```
 আমাদেরকে ফাঙ্কশন টি ব্যবহার করতে হবে যার মাধ্যমে wp-ajax.php সম্পর্ক তৈরি করে দিবো  <br>
 ```
 wp_localize_script( 'lmfwppt-scripts (handle)', 'lmfwppt_params (objectname)',
         	array(
         	    'nonce' => wp_create_nonce( 'lmwppt_nonce' ),
         	    'ajaxurl' => admin_url( 'admin-ajax.php' ),
         	)
         );
 
 ```
 
 wp_localize_script অনেক গুলি প্যারামিটার থাকে তার মধ্যে আমরা দুইটি পারমিটের use করবো <br>
 আমাদের যে স্ক্রিপ্ট ফাইলটি আছে ওইটা add করবো প্রথম প্যারামিটার আর দ্বিতীয় প্যারামিটার তৃতীয় প্যারামিটার আমরা একটি array pass করবো <br>
 
 
 # আজকে আমরা জানবো কিভাবে wp Delete করতে হয় ❤
 ```
 $actions['delete'] = sprintf( '<a href="%s" class="submitdelete" onclick="return confirm(\'Are you sure?\');" title="%s">%s</a>', wp_nonce_url( admin_url( 'admin-post.php?action=lmfwppt-delete-license&id=' . $item->id ), 'lmfwppt-delete-license' ), $item->id, __( 'Delete', 'license-manager-wppt' ), __( 'Delete', 'license-manager-wppt' ) );

 wp_nonce_url( admin_url(
    'admin-post.php?action=wd-ac-delete-address&id=' . $item->id ) 
'wd-ac-delete-address' )
 
 ```
 ✔ wp_nonce_url দুইটি প্যারামিটার পাস করে 
 ✔ প্রথমটি url 
 ✔ দ্বিতীয়টি action 
 
 ```
 // Delete License Id
    function lmfwppt_delete_license( $id ) {
        global $wpdb;

        return $wpdb->delete(
            $wpdb->prefix . 'lmfwppt_licenses',
            [ 'id' => $id ],
            [ '%d' ]
        );
    }


    function __construct() {
        add_action( 'admin_init', [ $this, 'delete_license' ] );
    }

    // Get The Action
    function delete_license() {
        
    if( isset( $_REQUEST['action'] ) && $_REQUEST['action'] == "lmfwppt-delete-license" ){
        if ( ! wp_verify_nonce( $_REQUEST['_wpnonce'], 'lmfwppt-delete-license' ) ) {
            wp_die( 'Are you cheating?' );
        }

        if ( ! current_user_can( 'manage_options' ) ) {
            wp_die( 'Are you cheating?' );
        }

        $id = isset( $_REQUEST['id'] ) ? intval( $_REQUEST['id'] ) : 0; 

        if ( $this->lmfwppt_delete_license( $id ) ) {
            $redirected_to = admin_url( 'admin.php?page=license-manager-wppt-licenses&deleted=true' );
        } else {
            $redirected_to = admin_url( 'admin.php?page=license-manager-wppt-licenses&deleted=false' );
        }

        wp_redirect( $redirected_to );
        exit;

        }    
    } 
 
 ```
 # আজকে আমরা জানবো কি ভাবে প্লাগিন স্ট্রাকচার তৈরি করতে হয় 
🎁 বেসিক কাজটি সম্পর্ণ করবো। <br>
🎁 wp-admin এবং wp-includes ফোল্ডার দুই টি অ্যাড করে নিবো। কারণ প্লাগিন অনেক ফাঙ্কশন আছে যে গুলি আমরা সহজেই পেয়ে যাই। আমরা যেই টাকে বলি code-combination feature .<br>
🎁 প্লাগিন নামটি হবে main ফোল্ডার নামে। ex:মনে করি ফোল্ডার নাম first-plugin তাহলে ফাইলের নাম হবে first-plugin.php (সবাই ফলো করে ) <br>
 
❓❓❓❓❓❓ # প্লাগিন মূল কাজটি শুরু ❓❓❓❓❓❓❓❓ <br>
👍 প্লাগিন header টি add করে নিবো। <br>
👍 প্লাগিন মেনু থেকে প্লাগিন টাকে active করবো <br>

🤦‍♀️🤦‍♀️🤦‍♀️🤦‍♀️ class o function লিখার সময় doc লিখবো 🤦‍♀️🤦‍♀️🤦‍♀️🤦‍♀️

# ক্লাস সম্পর্কে জানবো 

আমরা অনেক সময় ক্লাস সামনে final লিখে দেখি। 
final টি হচ্ছে একটি কীওয়ার্ড। 
final কীওয়ার্ড দেয়ার মাধ্যমে ওই class টা কে overriden করতে পারবে না। 


✌ ক্লাস ভিতরে যে ফাঙ্কশন গুলি লিখে হয় তাদের আমরা বলি মেথড <br>
✌ php তে অনেক গুলি magic মেথড রয়েছে 

# construct মেথড সম্পর্কে জানবো <br>
❤ construct method একটি magic method .<br>
❤ construct method রিটার্ন করে যাই না। <br>
❤ construct method প্যারামিটার পাস করে যাই। <br>
❤ construct method লিখার সময় অবশ্যই ( __ ) দিতে হবে। <br>
❤ construct method কে আলাদা করে রিটার্ন করতে হয় না। <br>
❤ যখন ক্লাস লিখি তখন বা তার সাথে সাথে construct মেথড টি call হয়। <br>
```
            private function __construct() {
		// Loaded textdomain
		add_action('plugins_loaded', array( $this, 'plugin_loaded_action' ), 10, 2);

		// Enqueue frontend scripts
		add_action( 'wp_enqueue_scripts', array( $this, 'enqueue_scripts' ) );
		add_action( 'admin_enqueue_scripts', array( $this, 'admin_scripts' ), 100 );

		// Added plugin action link
		add_filter( 'plugin_action_links_' . plugin_basename( __FILE__ ), array( $this, 'action_links' ) );

		// trigger upon plugin activation/deactivation
		register_activation_hook( __FILE__, array( $this, 'plugin_activation' ) );
		//register_deactivation_hook( __FILE__, array( $this, 'plugin_deactivation' ) );

	}

```
কনস্ট্রাক্ট মেথড সামনে private ব্যবহার করলে ওই ক্লাস টাকে কেউ initialize করতে পারবে না।  

# static মেথড সম্পর্কে জানবো  
STATIC METHOD : স্ট্যাটিক ফাংশন ক্লাসের সাথে যুক্ত, ক্লাসের উদাহরণ নয়। তাদের কেবল স্থির পদ্ধতি এবং স্ট্যাটিক ভেরিয়েবল অ্যাক্সেস করার অনুমতি দেওয়া হয়েছে।

স্ট্যাটিক মেথড ex :
```
     public static function init(){
     	static $instance = false;

        if ( ! $instance ) {
            $instance = new self();
        }

        return $instance;
	}

```
যখন কেউ ক্লাসটাকে initialize করবে তখন শুধু init মেথডকে কল করবে। 
এখন আমার একটি নতুন ফাঙ্কশন তৈরি করবো যা ফাঙ্কশন কাজ হবে ক্লাস নতুন একটি instance তৈরি করবে আর ওই instnace কে return করবে। 

ফাঙ্কশন টি দেওয়া হলো। 

```
/**
 * Initialize plugin
 */
function lmfwpptwcext(){
	return LMFWPPTWCEXT::init();
}

// Let's start it
lmfwpptwcext();

```
 
 

