# Woocommerce custom script

useful page https://businessbloomer.com/woocommerce-visual-hook-guide-single-product-page/

# Adding the new custom field on the WooCommerce checkout page (date)
        <?php

        add_action( 'woocommerce_after_checkout_billing_form', 'display_extra_fields_after_billing_address' , 10, 1 );
        function display_extra_fields_after_billing_address () {
             _e( "Installation Date: ", "add_extra_fields");
             ?>
        <br>
        <input type="text" name="add_installation_date" class="add_installation_date" placeholder="Installation Date" readonly="readonly">
        <script>
        jQuery(document).ready(function($) {
            $(".add_installation_date").datepicker({
                minDate: 0,
                dateFormat: 'yy-mm-dd',
            });
        });
        </script>
        <?php 
        }
        add_action( 'wp_enqueue_scripts', 'enqueue_datepicker' );
        function enqueue_datepicker() {
            if ( is_checkout() ) {
                // Load the datepicker script (pre-registered in WordPress).
                wp_enqueue_script( 'jquery-ui-datepicker' );
                // You need styling for the datepicker. For simplicity I've linked to Google's hosted jQuery UI CSS.
                wp_register_style( 'jquery-ui', '//code.jquery.com/ui/1.11.2/themes/smoothness/jquery-ui.css' );
                wp_enqueue_style( 'jquery-ui' );  
            }
        }
        add_action( 'woocommerce_checkout_update_order_meta', 'add_order_delivery_date_to_order' , 10, 1);
        function add_order_delivery_date_to_order ( $order_id ) {
             if ( isset( $_POST ['add_installation_date'] ) &&  '' != $_POST ['add_installation_date'] ) {
                 add_post_meta( $order_id, '_delivery_date',  sanitize_text_field( $_POST ['add_installation_date'] ) );
             }
        }
        add_filter( 'woocommerce_email_order_meta_fields', 'add_installation_date_to_emails' , 10, 3 );
        function add_installation_date_to_emails ( $fields, $sent_to_admin, $order ) {
            if( version_compare( get_option( 'woocommerce_version' ), '3.0.0', ">=" ) ) {            
                $order_id = $order->get_id();
            } else {
                $order_id = $order->id;
            }
            $delivery_date = get_post_meta( $order_id, '_delivery_date', true );
            if ( '' != $delivery_date ) {
                $fields[ 'Installation Date' ] = array(
                'label' => __( 'Installation Date', 'add_extra_fields' ),
                'value' => $delivery_date,
                );
             }
            return $fields;
        }
        
        //add info in order received data. Ex: installatino date or tracking no
        add_filter( 'woocommerce_order_details_after_order_table', 'add_installation_date_to_order_received_page', 10 , 1 );
        function add_installation_date_to_order_received_page ( $order ) {
            if( version_compare( get_option( 'woocommerce_version' ), '3.0.0', ">=" ) ) {            
                $order_id = $order->get_id();
            } else {
                $order_id = $order->id;
            }
            $delivery_date = get_post_meta( $order_id, '_delivery_date', true );

            if ( '' != $delivery_date ) {
                echo '<p><strong>' . __( 'Installation Date', 'add_extra_fields' ) . ':</strong> ' . $delivery_date;
            }
        }


        //display date in woocommercer edit order page
        function add_display_order_data_in_admin( $order ){  ?>
        <div class="order_data_column">
            <h4>
                <?php _e( 'Additional Information', 'woocommerce' ); ?><a href="#" class="edit_address">
                    <?php _e( 'Edit', 'woocommerce' ); ?></a></h4>
            <div class="address">
                <?php
                    echo '<p><strong>' . __( 'Installtion date' ) . ':</strong>' . get_post_meta( $order->id, '_delivery_date', true ) . '</p>';
                     ?>
            </div>
            <div class="edit_address">
                <?php woocommerce_wp_text_input( array( 'class' => 'date-picker','id' => '_delivery_date', 'label' => __( 'Installation date' ), 'wrapper_class' => '_billing_company_field', 'placeholder' => 'Installation date' , 'custom_attributes' => ['style' => 'width:100%!important;'] ) ); ?>
            </div>
        </div>
        <?php }
        add_action( 'woocommerce_admin_order_data_after_order_details', 'add_display_order_data_in_admin' );



        function save_extra_details( $post_id, $post ){
            update_post_meta( $post_id, '_delivery_date', wc_clean( $_POST[ '_delivery_date' ] ) );
        }
        add_action( 'woocommerce_process_shop_order_meta', 'save_extra_details', 45, 2 );

ref : https://rudrastyh.com/woocommerce/customize-order-details.html



# change select option in archive page (shop page)  
Change "Select option" to "Order now"  

        //change select option in archive page (shop page)
        add_filter( 'add_to_cart_text', 'woo_custom_product_add_to_cart_text' );            // < 2.1
        add_filter( 'woocommerce_product_add_to_cart_text', 'woo_custom_product_add_to_cart_text' );  // 2.1 +

        function woo_custom_product_add_to_cart_text() {

            return __( 'Order now', 'woocommerce' );
        }
        

# Custom Payment Gateway

        <?php
        /*
        Plugin Name: Custom Payment Gateway
        Description: Custom payment gateway example
        Author: Lafif Astahdziq
        Author URI: https://lafif.me
        */

        if ( ! defined( 'ABSPATH' ) ) {
            exit; // Exit if accessed directly
        }

        /**
         * Custom Payment Gateway.
         *
         * Provides a Custom Payment Gateway, mainly for testing purposes.
         */
        add_action('plugins_loaded', 'init_custom_gateway_class');
        function init_custom_gateway_class(){

            class WC_Gateway_Custom extends WC_Payment_Gateway {

                public $domain;

                /**
                 * Constructor for the gateway.
                 */
                public function __construct() {

                    $this->domain = 'custom_payment';

                    $this->id                 = 'custom';
                    $this->icon               = apply_filters('woocommerce_custom_gateway_icon', '');
                    $this->has_fields         = false;
                    $this->method_title       = __( 'Custom', $this->domain );
                    $this->method_description = __( 'Allows payments with custom gateway.', $this->domain );

                    // Load the settings.
                    $this->init_form_fields();
                    $this->init_settings();

                    // Define user set variables
                    $this->title        = $this->get_option( 'title' );
                    $this->description  = $this->get_option( 'description' );
                    $this->instructions = $this->get_option( 'instructions', $this->description );
                    $this->order_status = $this->get_option( 'order_status', 'completed' );

                    // Actions
                    add_action( 'woocommerce_update_options_payment_gateways_' . $this->id, array( $this, 'process_admin_options' ) );
                    add_action( 'woocommerce_thankyou_custom', array( $this, 'thankyou_page' ) );

                    // Customer Emails
                    add_action( 'woocommerce_email_before_order_table', array( $this, 'email_instructions' ), 10, 3 );
                }

                /**
                 * Initialise Gateway Settings Form Fields.
                 */
                public function init_form_fields() {

                    $this->form_fields = array(
                        'enabled' => array(
                            'title'   => __( 'Enable/Disable', $this->domain ),
                            'type'    => 'checkbox',
                            'label'   => __( 'Enable Custom Payment', $this->domain ),
                            'default' => 'yes'
                        ),
                        'title' => array(
                            'title'       => __( 'Title', $this->domain ),
                            'type'        => 'text',
                            'description' => __( 'This controls the title which the user sees during checkout.', $this->domain ),
                            'default'     => __( 'Custom Payment', $this->domain ),
                            'desc_tip'    => true,
                        ),
                        'order_status' => array(
                            'title'       => __( 'Order Status', $this->domain ),
                            'type'        => 'select',
                            'class'       => 'wc-enhanced-select',
                            'description' => __( 'Choose whether status you wish after checkout.', $this->domain ),
                            'default'     => 'wc-completed',
                            'desc_tip'    => true,
                            'options'     => wc_get_order_statuses()
                        ),
                        'description' => array(
                            'title'       => __( 'Description', $this->domain ),
                            'type'        => 'textarea',
                            'description' => __( 'Payment method description that the customer will see on your checkout.', $this->domain ),
                            'default'     => __('Payment Information', $this->domain),
                            'desc_tip'    => true,
                        ),
                        'instructions' => array(
                            'title'       => __( 'Instructions', $this->domain ),
                            'type'        => 'textarea',
                            'description' => __( 'Instructions that will be added to the thank you page and emails.', $this->domain ),
                            'default'     => '',
                            'desc_tip'    => true,
                        ),
                    );
                }

                /**
                 * Output for the order received page.
                 */
                public function thankyou_page() {
                    if ( $this->instructions )
                        echo wpautop( wptexturize( $this->instructions ) );
                }

                /**
                 * Add content to the WC emails.
                 *
                 * @access public
                 * @param WC_Order $order
                 * @param bool $sent_to_admin
                 * @param bool $plain_text
                 */
                public function email_instructions( $order, $sent_to_admin, $plain_text = false ) {
                    if ( $this->instructions && ! $sent_to_admin && 'custom' === $order->payment_method && $order->has_status( 'on-hold' ) ) {
                        echo wpautop( wptexturize( $this->instructions ) ) . PHP_EOL;
                    }
                }

                public function payment_fields(){

                    if ( $description = $this->get_description() ) {
                        echo wpautop( wptexturize( $description ) );
                    }

                    ?>
                    <div id="custom_input">
                        <p class="form-row form-row-wide">
                            <label for="mobile" class=""><?php _e('Mobile Number', $this->domain); ?></label>
                            <input type="text" class="" name="mobile" id="mobile" placeholder="" value="">
                        </p>
                        <p class="form-row form-row-wide">
                            <label for="transaction" class=""><?php _e('Transaction ID', $this->domain); ?></label>
                            <input type="text" class="" name="transaction" id="transaction" placeholder="" value="">
                        </p>
                    </div>
                    <?php
                }

                /**
                 * Process the payment and return the result.
                 *
                 * @param int $order_id
                 * @return array
                 */
                public function process_payment( $order_id ) {

                    $order = wc_get_order( $order_id );

                    $status = 'wc-' === substr( $this->order_status, 0, 3 ) ? substr( $this->order_status, 3 ) : $this->order_status;

                    // Set order status
                    $order->update_status( $status, __( 'Checkout with custom payment. ', $this->domain ) );

                    // Reduce stock levels
                    $order->reduce_order_stock();

                    // Remove cart
                    WC()->cart->empty_cart();

                    // Return thankyou redirect
                    return array(
                        'result'    => 'success',
                        'redirect'  => $this->get_return_url( $order )
                    );
                }
            }
        }

        add_filter( 'woocommerce_payment_gateways', 'add_custom_gateway_class' );
        function add_custom_gateway_class( $methods ) {
            $methods[] = 'WC_Gateway_Custom'; 
            return $methods;
        }

        add_action('woocommerce_checkout_process', 'process_custom_payment');
        function process_custom_payment(){

            if($_POST['payment_method'] != 'custom')
                return;

            if( !isset($_POST['mobile']) || empty($_POST['mobile']) )
                wc_add_notice( __( 'Please add your mobile number', $this->domain ), 'error' );


            if( !isset($_POST['transaction']) || empty($_POST['transaction']) )
                wc_add_notice( __( 'Please add your transaction ID', $this->domain ), 'error' );

        }

        /**
         * Update the order meta with field value
         */
        add_action( 'woocommerce_checkout_update_order_meta', 'custom_payment_update_order_meta' );
        function custom_payment_update_order_meta( $order_id ) {

            if($_POST['payment_method'] != 'custom')
                return;

            update_post_meta( $order_id, 'mobile', $_POST['mobile'] );
            update_post_meta( $order_id, 'transaction', $_POST['transaction'] );
        }

        /**
         * Display field value on the order edit page
         */
        add_action( 'woocommerce_admin_order_data_after_billing_address', 'custom_checkout_field_display_admin_order_meta', 10, 1 );
        function custom_checkout_field_display_admin_order_meta($order){
            $method = get_post_meta( $order->id, '_payment_method', true );
            if($method != 'custom')
                return;

            $mobile = get_post_meta( $order->id, 'mobile', true );
            $transaction = get_post_meta( $order->id, 'transaction', true );

            echo '<p><strong>'.__( 'Mobile Number' ).':</strong> ' . $mobile . '</p>';
            echo '<p><strong>'.__( 'Transaction ID').':</strong> ' . $transaction . '</p>';
        }




        /**
         * display date in woocommercer edit order page
         */
        function add_display_order_data_in_admin( $order ){  ?>

        <div class="order_data_column">
            <h4>
                <?php _e( 'Additional Information', 'woocommerce' ); ?>
                <a href="#" class="edit_address">
                    <?php _e( 'Edit', 'woocommerce' ); ?>

                </a>
            </h4>
            <div class="address">
                <?php
                    echo '<p><strong>' . __( 'Mobile Number' ) . ':</strong>' . get_post_meta( $order->id, 'mobile', true ) . '</p>';
                 ?>
            </div>
            <div class="edit_address">
                <?php 
                woocommerce_wp_text_input( array( 'class' => 'mobile',
                                                  'id' => 'mobile', 
                                                  'label' => __( 'Installation date' ), 
                                                  'wrapper_class' => '_billing_company_field', 
                                                  'placeholder' => 'mobile' , 
                                                  'custom_attributes' => ['style' => 'width:100%!important;'] ) );
                                                  ?>
            </div>
        </div>
        <?php }
        add_action( 'woocommerce_admin_order_data_after_order_details', 'add_display_order_data_in_admin' );




        function woo_edit_custom_general_fields_save( $ord_id ){
            update_post_meta( $ord_id, 'mobile', wc_clean( $_POST[ 'mobile' ] ) );
            // wc_clean() and wc_sanitize_textarea() are WooCommerce sanitization functions
        }
        add_action( 'woocommerce_process_shop_order_meta', 'woo_edit_custom_general_fields_save' );

# Add a Link Back to the Order in WooCommerce New Order Notifications Email

        // Add a Link Back to the Order in WooCommerce New Order Notifications Email
        add_action( 'woocommerce_email_after_order_table', 'add_link_back_to_order', 10, 2 );

        function add_link_back_to_order( $order, $is_admin ) {

            $order = new WC_Order($order->get_id());
            $order_key = $order->get_order_key();

            if ( $order->has_status( 'processing' ) || $order->has_status( 'completed' )) {
              //link for customer view thank you page to upload payment slip
              $link = '<h2><a href="'.get_site_url().'/checkout/order-received/'.absint( $order->get_id() ).'/?key='.$order_key.'" >'.__( 'Klik sini untuk ke page order').'</a></h2></br></br>';

              if ($is_admin ) {
                  //link for admin to view woocommerce order ini admin dashboard
                  $link .= '<p>';
                  $link .= '<a href="'. admin_url( 'post.php?post=' . absint( $order->get_id() ) . '&action=edit' ) .'" >';
                  $link .= __( 'Click here to go to the order page');
                  $link .= '</a>';
                  $link .= '</p></br></br>';
              }

              // Return the link into the email
              echo $link;
              }

        }
        
# add short description to email notifications
        
        // add short description to email notifications
        add_action( 'woocommerce_order_item_meta_end', 'product_description_in_new_email_notification', 10, 4 );
        function product_description_in_new_email_notification( $item_id, $item, $order = null, $plain_text = false ){
            $product = $item->get_product();

            // Handling product variations
            if( $product->is_type('variation') )
                $product = wc_get_product( $item->get_product_id() );

            // Display the product short description
            echo '<div class="product-description" style="margin:10px 0 0;"><p>' . $product->get_short_description() . '</p></div>';
        }

# added by izzul - for kewangan puspanita verify order. only can view woocommerce orders
        
        // added by izzul - for kewangan puspanita verify order. only can view woocommerce orders
        add_action( 'admin_menu', 'remove_menu_pages', 999);
        function remove_menu_pages() {
          global $current_user;

          $user_roles = $current_user->roles;
          $user_role = array_shift($user_roles);
          if($user_role == "kewangan_puspanita") {
            // remove_menu_page( 'themes.php' );
            // remove_submenu_page( 'options-general.php','options-media.php' );
            remove_submenu_page('woocommerce', 'wc-settings'); //hide woocommerce setting
            remove_submenu_page('woocommerce', 'wc-addons'); //hide woocommerce addon
            remove_submenu_page('woocommerce', 'wcj-tools');//hide booster tools
            remove_submenu_page('woocommerce', 'wc-status');//hide woocommerce status
          }
        }

# lock status dropdown in edit order page - to avoid user playaround with status and mess up flow

        // lock if status == cancelled
        if(jQuery( "#order_status option:selected" ).val()=="wc-cancelled"){ jQuery('#order_status').select2("enable", false); }
        
        // lock if status == Completed
        if(jQuery( "#order_status option:selected" ).val()=="wc-completed"){ jQuery('#order_status').select2("enable", false); }
        
        
# Display Variations' Stock @ WooCommerce Shop

        add_action( 'woocommerce_after_shop_loop_item', 'izoolz_show_stock_variations_loop' );
    
        function izoolz_show_stock_variations_loop(){
                echo '<center><div>';
            global $product;
            if ( $product->get_type() == 'variable' ) {
                foreach ( $product->get_available_variations() as $key ) {
                    $attr_string = array();
                    foreach ( $key['attributes'] as $attr_name => $attr_value ) {
                        $attr_string[] = $attr_value;
                    }
                    if ( $key['max_qty'] > 0 ) { 
                      echo '<span style="margin-left:-4px;padding: 5px 5px;text-transform: uppercase;font-size:8px;background-color:#CCCCCC;">'.implode( ', ', $attr_string ).'</span> '; 
                    } else { 
                      echo '<span style="margin-left:-3px;color:white;text-transform: uppercase;font-size:8px;text-decoration: line-through;background-color:red;padding:5px 5px;">'.implode(', ', $attr_string ).'</span>'; 
                    }
                }
            }
            echo '</div></center>';
        }
        
# Display Variations' Stock @ WooCommerce single page product

        add_action( 'woocommerce_before_add_to_cart_button', 'izoolz_show_variant_at_single_page', 20 );
 
        function izoolz_show_variant_at_single_page() {
                global $product;

                if ( $product->is_type( 'variable' ) ) {
                        echo '<div style="font-size:10px">Please choose at the dropdown above.</div>';
                        echo '<div style="margin-bottom:30px;overflow: auto;">';
                        $variations = $product->get_available_variations();
                        $display_att ='';
                        foreach ($variations as $key => $value){
                                $scs_wc_size = $value['attributes']['attribute_pa_size'];
                                if ( $value['max_qty'] > 0 ) { 
                      echo '<span style="padding: 0px 10px;text-transform: uppercase;font-size:15px;background-color:#CCCCCC;float:left;border: 1px solid white">'.$scs_wc_size.'</span> '; 
                    } else{
                                        echo  '<span style="padding: 0px 10px;color:white;text-transform: uppercase;font-size:15px;text-decoration: line-through;background-color:red;float:left;border: 1px solid white;">'.$scs_wc_size.'</span>';
                                }
                        }
                        echo '</div>';
                }
        }

# Add link to Order Received page in Email - useful for guest with no registered account

        // Add a Link Back to the Order in WooCommerce New Order Notifications Email
        add_action( 'woocommerce_email_order_details', 'add_link_back_to_order', 10, 2 );
        function add_link_back_to_order( $order, $is_admin ) {

            $order_info = new WC_Order($order->get_id());
            $order_key = $order->get_order_key();
            $link = '';

            if ( $order_info->has_status( 'processing' ) || $order_info->has_status( 'completed' )) {

                  if ($is_admin ) {
                      //link for admin to view woocommerce order ini admin dashboard
                      $link .= '<p>';
                      $link .= '<a href="'. admin_url( 'post.php?post=' . absint( $order_info->get_id() ) . '&action=edit' ) .'" >';
                      $link .= __( 'Click here to manage the order.');
                      $link .= '</a>';
                      $link .= '</p></br></br>';
                  } else {
                    //link for customer view thank you page to upload payment slip
                      $link .= '<p>';
                      $link .= '<a href="'.get_site_url().'/checkout/order-received/'.absint( $order_info->get_id() ).'/?key='.$order_key.'" >';
                      $link .= __( 'Click here to see your order details in web browser.');
                      $link .= '</a>';
                      $link .= '</p></br></br>';
                  }

              echo $link;

              }

        }
        
Visual email hook: https://businessbloomer.com/woocommerce-visual-hook-guide-emails/
https://businessbloomer.com/category/woocommerce-tips/visual-hook-series/

# Add custom field in \_tracking in _Order receive page_ . In this case, show tracking no to customer

In this case, \_tracking is generated by ACF

        add_filter( 'woocommerce_order_details_after_order_table', 'add_trackking_no_to_order_received_page', 10 , 1 );
        function add_trackking_no_to_order_received_page ( $order ) {

            $order_info = $order->get_id();

            if ($order->has_status( 'completed' )) {

                $tracking_no = get_post_meta( $order_info, '_tracking', true );

                if ( $tracking_no ) {
                    echo '<p><strong>' . __( 'Your Tracking No', 'add_extra_fields' ) . ':</strong> ' . $tracking_no;
                } else {
                     echo '<p><strong>' . __( 'Your Tracking No', 'add_extra_fields' ) . ':</strong> No tracking number is provided';
                }
            }
        }

# Admin page order preview.

        // Add custom order meta data to make it accessible in Order preview template
        add_filter( 'woocommerce_admin_order_preview_get_order_details', 'admin_order_preview_add_custom_meta_data', 10, 2 );
        function admin_order_preview_add_custom_meta_data( $data, $order ) {
            // Replace '_custom_meta_key' by the correct postmeta key
            if( $custom_value = $order->get_meta('_custom_meta_key') )
                $data['custom_key'] = $custom_value; // <= Store the value in the data array.

            return $data;
        }

        // Display custom values in Order preview
        add_action( 'woocommerce_admin_order_preview_end', 'custom_display_order_data_in_admin' );
        function custom_display_order_data_in_admin(){
            // Call the stored value and display it
            echo '<div>Value: {{data.custom_key}}</div><br>';
        }
        
# allow shop manager to change user role

reference https://woocommerce.wordpress.com/2018/10/11/woocommerce-3-4-6-security-fix-release-notes/

        function myextension_shop_manager_role_edit_capabilities( $roles ) {
                $roles = ['dropship','subscriber'];
            return $roles;
        }
        add_filter( 'woocommerce_shop_manager_editable_roles', 'myextension_shop_manager_role_edit_capabilities' ); 
        
# DIVI search menu only search products

add to WP Admin > Divi > Theme Options > Integration > Body Code:

        <script>
            jQuery(function($){
                $( ".et-search-form" ).append( '<input type="hidden" name="post_type" value="product" />' );
            });
        </script>
