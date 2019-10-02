# Woocommerce custom script

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

        $test_order = new WC_Order($order->id);
        $test_order_key = $test_order->order_key;

        if ( $order->has_status( 'on-hold' ) || $order->has_status( 'completed' )) {
          //link for customer view thank you page to upload payment slip
          $link = '<h2><a href="'.get_site_url().'/checkout/order-received/'.absint( $order->id ).'/?key='.$test_order_key.'" >'.__( 'Klik sini untuk upload bukti bayaran ', 'shop.puspanita.org.my' ).'</a></h2></br></br>';

          if ($is_admin ) {
          //link for admin to view woocommerce order ini admin dashboard
          $link .= '<p>';
          $link .= '<a href="'. admin_url( 'post.php?post=' . absint( $order->id ) . '&action=edit' ) .'" >';
          $link .= __( 'Click here to go to the order page', 'shop.puspanita.org.my' );
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
            $remove_submenu = remove_submenu_page('woocommerce', 'wc-settings'); //hide woocommerce setting
            $remove_submenu = remove_submenu_page('woocommerce', 'wc-addons'); //hide woocommerce addon
            $remove_submenu = remove_submenu_page('woocommerce', 'wcj-tools');//hide booster tools
            $remove_submenu = remove_submenu_page('woocommerce', 'wc-status');//hide woocommerce status
          }
        }
