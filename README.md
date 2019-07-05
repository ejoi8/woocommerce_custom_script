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




# change select option in archive page (shop page)
        //change select option in archive page (shop page)
        add_filter( 'add_to_cart_text', 'woo_custom_product_add_to_cart_text' );            // < 2.1
        add_filter( 'woocommerce_product_add_to_cart_text', 'woo_custom_product_add_to_cart_text' );  // 2.1 +

        function woo_custom_product_add_to_cart_text() {

            return __( 'Order now', 'woocommerce' );
        }
