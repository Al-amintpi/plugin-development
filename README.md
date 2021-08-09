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
## আজকে আমরা জানবো কিভাবে woocommerce meta box তৈরি করতে হয়
আমি license নামে একটি metabox তৈরি করলাম <br>

<img src="/Downloads/download.png" alt="My cool logo"/>

```
/*
    * 
	 * Custom Meta Box And Tab
	 */
	add_filter('woocommerce_product_data_tabs', 'lmfwpptwcext_product_settings_tabs' );
	function lmfwpptwcext_product_settings_tabs( $tabs ){
	 
	    $tabs['lmfwpptwcext'] = array(
	        'label'    => __('License Manager', 'lmfwpptwcext'),
	        'target'   => 'lmfwpptwcext_product_data',
	        'class'   => 'show_if_simple',
	        'priority' => 21,
	    );
	    return $tabs;
	 
	}
	 
	/*
	 * Tab content
	 */
	add_action( 'woocommerce_product_data_panels', 'lmfwpptwcext_product_panels' );
	function lmfwpptwcext_product_panels(){

		$post_id = get_the_ID();
	 
	    echo '<div id="lmfwpptwcext_product_data" class="lmfwpptwcext_product_data panel woocommerce_options_panel">';
	 
	    woocommerce_wp_checkbox( array(
	        'id'           => 'active_license_management',
	        'class'       => 'lmfwpptwcext_checkbox',
	        'value'        => get_post_meta( $post_id, 'active_license_management', true ),
	        'label'        => __('Active License Management', 'lmfwpptwcext')
	    ) );

		    echo '<div class="lmfwpptwcext_product_fields">';

			    woocommerce_wp_select( array(
			        'id'          => 'select_product_type',
			        'class'       => 'select_product_type',
			        'value'       => get_post_meta( $post_id, 'select_product_type', true ),
			        'label'       => __('Select Products Type', 'lmfwpptwcext'),
			        'options'     => array( '' => 'Please select', 'theme' => 'Theme', 'plugin' => 'Plugin'),
			    ) );

			    woocommerce_wp_select( array(
			        'id'          => 'theme_product_list',
			        'class'          => 'select_product_list theme_product_list',
			        'value'       => get_post_meta( $post_id, 'theme_product_list', true ),
			        'label'       => __('Select Product', 'lmfwpptwcext'),
			        'options'     => lmfwpptwcext_generate(lmfwppt_get_product_list("theme")),
			    ) );

			    woocommerce_wp_select( array(
			        'id'          => 'plugin_product_list',
			        'class'          => 'select_product_list plugin_product_list',
			        'value'       => get_post_meta( $post_id, 'plugin_product_list', true ),
			        'label'       => __('Select Product', 'lmfwpptwcext'),
			        'options'     => lmfwpptwcext_generate(lmfwppt_get_product_list("plugin")),
			    ) );

			    woocommerce_wp_select( array(
			        'id'          => 'select_package',
			        'class'          => 'select_package',
			        'value'       => get_post_meta( $post_id, 'select_package', true ),
			        'label'       => __('Select Package', 'lmfwpptwcext'),
			        'options'     => array( '' => 'Please select'),
			    ) );
		 
		    echo '</div>';
	    echo '</div>';

	}
	

	/*
	*
	* Custom Meta box save
	*/
	add_action( 'woocommerce_process_product_meta', 'lmfwpptwcext_save_fields', 10, 2 );
	function lmfwpptwcext_save_fields( $id, $post ){
		
		update_post_meta( $id, 'active_license_management', $_POST['active_license_management'] );
		
		if( !empty( $_POST['select_product_type'] ) ) {
			update_post_meta( $id, 'select_product_type', $_POST['select_product_type'] );
		} 

		if( !empty( $_POST['theme_product_list'] ) ) {
			update_post_meta( $id, 'theme_product_list', $_POST['theme_product_list'] );
		}

		if( !empty( $_POST['plugin_product_list'] ) ) {
			update_post_meta( $id, 'plugin_product_list', $_POST['plugin_product_list'] );
		}
	 
		if( !empty( $_POST['select_package'] ) ) {
			update_post_meta( $id, 'select_package', $_POST['select_package'] );
		}  
    }


    /*
    *
    * product list select option pass
    */
	function lmfwpptwcext_generate( $data_arr ){
		$return = array();
		$return[''] = __('Select Product','textdomain');
		foreach ( $data_arr as $data ) {
			$return[$data->id] = $data->name;
		}
		return $return;
	}

```

## এখন আমরা জানবো কিভাবে variations তৈরি করতে হয় <br>

code example
```
/*
	*
	* custom field on product Add Variations
	*/
	add_action( 'woocommerce_variation_options_pricing', 'bbloomer_add_custom_field_to_variations', 10, 3 );
	function bbloomer_add_custom_field_to_variations( $loop, $variation_data, $variation ) {
	 
	   $post_id = $variation->ID;

		echo '<div id="lmfwpptwcext_product_data" class="lmfwpptwcext_product_data panel woocommerce_options_panel">';
		 
		    woocommerce_wp_checkbox( array(
		        'id'          => 'active_license_management[' . $loop . ']',
		        'class'       => 'lmfwpptwcext_checkbox',
		        'value'       => get_post_meta( $post_id, 'active_license_management', true ),
		        'label'       => __('Active License Management', 'lmfwpptwcext'),
		    ) );

		    echo '<div class="lmfwpptwcext_product_fields">';

			    woocommerce_wp_select( array(
			        'id'          => 'select_product_type[' . $loop . ']',
			        'class'          => 'select_product_type',
			        'value'       => get_post_meta( $post_id, 'select_product_type', true ),
			        'label'       => __('Select Products Type', 'lmfwpptwcext'),
			        'options'     => array( '' => 'Please select', 'theme' => 'Theme', 'plugin' => 'Plugin'),
			    ) );

			    woocommerce_wp_select( array(
			        'id'          => 'theme_product_list[' . $loop . ']',
			        'class'          => 'select_product_list theme_product_list',
			        'value'       => get_post_meta( $post_id, 'theme_product_list', true ),
			        'label'       => __('Select Product', 'lmfwpptwcext'),
			        'options'     => lmfwpptwcext_generate(lmfwppt_get_product_list("theme")),
			    ) );

			    woocommerce_wp_select( array(
			        'id'          => 'plugin_product_list[' . $loop . ']',
			        'class'          => 'select_product_list plugin_product_list',
			        'value'       => get_post_meta( $post_id, 'plugin_product_list', true ),
			        'label'       => __('Select Product', 'lmfwpptwcext'),
			        'options'     => lmfwpptwcext_generate(lmfwppt_get_product_list("plugin")),
			    ) );

			    woocommerce_wp_select( array(
			        'id'          => 'select_package[' . $loop . ']',
			        'class'       => 'select_package',
			        'value'       => get_post_meta( $post_id, 'select_package', true ),
			        'label'       => __('Select Package', 'lmfwpptwcext'),
			        'options'     => array( '' => 'Please select'),
			    ) );
			 
			    echo '</div>';
		    echo '</div>';
	}


	/*
	*
	* Save custom field on product variation save
	*/ 
	add_action( 'woocommerce_save_product_variation', 'bbloomer_save_custom_field_variations', 10, 2 );
	function bbloomer_save_custom_field_variations( $variation_id, $i ) {

		update_post_meta( $variation_id, 'active_license_management', esc_attr( $_POST['active_license_management'][$i] ) );

		if( !empty( $_POST['select_product_type'][$i] ) ) {
			update_post_meta( $variation_id, 'select_product_type', esc_attr( $_POST['select_product_type'][$i] ) );
		}	

		if( !empty( $_POST['theme_product_list'][$i] ) ) {
			update_post_meta( $variation_id, 'theme_product_list', esc_attr( $_POST['theme_product_list'][$i] ) );
		}

		if( !empty( $_POST['plugin_product_list'][$i] ) ) {
			update_post_meta( $variation_id, 'plugin_product_list', esc_attr( $_POST['plugin_product_list'][$i] ) );
		}

		if( !empty( $_POST['select_package'][$i] ) ) {
			update_post_meta( $variation_id, 'select_package', esc_attr( $_POST['select_package'][$i] ) );
		}

	}



```

## এখন আমরা জানবো কিভাবে woocommerce My account tab কিভাবে মেনু অ্যাড করতে হয়

আমি license নামে একটি মেনু অ্যাড করবো 

```
/*
	*
	* Add A Menu on My Account menu tab
	*/
	add_filter ( 'woocommerce_account_menu_items', 'lmfwpptwcext_license_link', 40 );
	function lmfwpptwcext_license_link( $menu_links ){
		
		$menu_links = array_slice( $menu_links, 0, 5, true ) 
		+ array( 'license' => 'License' )
		+ array_slice( $menu_links, 5, NULL, true );
		
		return $menu_links;
	}

	add_action( 'init', 'lmfwpptwcext_add_endpoint' );
	function lmfwpptwcext_add_endpoint() {
		add_rewrite_endpoint( 'license', EP_PAGES );

	}

	add_action( 'woocommerce_account_license_endpoint', 'lmfwpptwcext_my_account_endpoint_content' );
	function lmfwpptwcext_my_account_endpoint_content() {
		echo 'Last time you logged in: yesterday from Safari.';
	}

```
## এখন আমরা জানবো কিভাবে cart ও order page Variations Data show করাতে হয় 

```
/**
	 * Add engraving text to cart item.
	 *
	 * @param array $cart_item_data
	 * @param int   $product_id
	 * @param int   $variation_id
	 *
	 * @return array
	 */
	add_filter( 'woocommerce_add_cart_item_data', 'iconic_add_engraving_text_to_cart_item', 10, 3 );
	function iconic_add_engraving_text_to_cart_item( $cart_item_data, $product_id, $variation_id ) {
		
		$product_id = isset( $variation_id ) && $variation_id != "0" ? sanitize_text_field( $variation_id ) : sanitize_text_field( $product_id );

		$is_active = get_post_meta( $product_id, 'active_license_management', true );
		if( !$is_active ) {
			return $cart_item_data;
		}else{
			$cart_item_data['is_active'] = $is_active;
		}

		$product_type = get_post_meta( $product_id, 'select_product_type', true );
		if( isset( $product_type ) ) {
			$cart_item_data['product_type'] = $product_type;
		}

		$theme_id = get_post_meta( $product_id, 'theme_product_list', true );
		if( isset( $theme_id ) ) {
			$cart_item_data['theme_id'] = $theme_id;
		}

		$plugin_id = get_post_meta( $product_id, 'plugin_product_list', true );
		if( isset( $plugin_id ) ) {
			$cart_item_data['plugin_id'] = $plugin_id;
		}

		$package_id = get_post_meta( $product_id, 'select_package', true );
		if( isset( $package_id ) ) {
			$cart_item_data['package_id'] = $package_id;
		}

		return $cart_item_data;
	}


	/*
	* Display custom data on cart and checkout page.
	*
	*/
	add_filter( 'woocommerce_get_item_data', 'get_item_data' , 25, 2 );
	function get_item_data ( $item_data, $cart_item ) {
	     
		$product_id = isset( $cart_item['variation_id'] ) && $cart_item['variation_id'] != "0" ? sanitize_text_field( $cart_item['variation_id'] ) : sanitize_text_field( $cart_item['product_id'] );

		$is_active = isset($cart_item['is_active']) ? sanitize_text_field($cart_item['is_active']) : null;

		if( !$is_active ) {
			return $item_data;
		}

		$product_type = isset( $cart_item['product_type'] ) ? sanitize_text_field( $cart_item['product_type'] ) : null;
		if ( $product_type ) {
			$item_data[] = array(
				'key'     => __( 'Product Type', 'lmfwpptwcext' ),
				'value'   =>  $product_type,
				'display' => '',
			);
		}

	    
	    if ( $product_type == "theme" && isset( $cart_item['theme_id'] ) && $cart_item['theme_id'] ) {
	    	$theme_name = LMFWPPT_ProductsHandler::get_product($cart_item['theme_id'])['name'];
		    $item_data[] = array(
				'key'     => __( 'Theme Name', 'lmfwpptwcext' ),
				'value'   => $theme_name,
				'display' => '',
			);
	    }
		
	     
		if ( $product_type == "plugin" && isset( $cart_item['plugin_id'] ) && $cart_item['plugin_id'] ) {
			$plugin_name = LMFWPPT_ProductsHandler::get_product($cart_item['plugin_id'])['name'];
				$item_data[] = array(
				'key'     => __( 'Plugin Name', 'textdomain' ),
				'value'   => $plugin_name,
				'display' => '',
			);
		}

		if( isset( $cart_item['package_id'] ) && $cart_item['package_id'] ) {
			$package_name = LMFWPPT_ProductsHandler::get_package_name($cart_item['package_id']);
			   
				$item_data[] = array(
				'key'     => __( 'Package', 'textdomain' ),
				'value'   => $package_name,
				'display' => '',
			);
		}

		return $item_data;
	}



```

## এখন আমরা জানবো কিভাবে Email Variations Data গুলি শো করতে হয় 

```

/**
	 * Add engraving text to order.
	 *
	 * @param WC_Order_Item_Product $item
	 * @param string                $cart_item_key
	 * @param array                 $values
	 * @param WC_Order              $order
	 */
	add_action( 'woocommerce_checkout_create_order_line_item', 'iconic_add_engraving_text_to_order_items', 10, 4 );
	function iconic_add_engraving_text_to_order_items( $item, $cart_item_key, $values, $order ) {

		$is_active = isset($values['is_active']) ? sanitize_text_field($values['is_active']) : null;

		if( !$is_active ) {
			return;
		}

		$product_type = isset( $values['product_type'] ) ? sanitize_text_field( $values['product_type'] ) : null;
		if($product_type){
			$item->add_meta_data( __( 'Product_type', 'iconic' ), $product_type );
		}

		if ( $product_type == "theme" && isset( $values['theme_id'] ) && $values['theme_id'] ){
			$theme_name = LMFWPPT_ProductsHandler::get_product($values['theme_id'])['name'];
			$item->add_meta_data( __( 'Theme Name', 'iconic' ), $theme_name );
		}

		if ( $product_type == "plugin" && isset( $values['plugin_id'] ) && $values['plugin_id'] ){
			$plugin_name = LMFWPPT_ProductsHandler::get_product($values['plugin_id'])['name'];
			$item->add_meta_data( __( 'Plugin Name', 'iconic' ), $plugin_name );
		}

		if( isset( $values['package_id'] ) && $values['package_id'] ){
			$package_name = LMFWPPT_ProductsHandler::get_package_name($values['package_id']);
			$item->add_meta_data( __( 'Package Name', 'iconic' ), $package_name );
		}
	}

``
 

