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
 

