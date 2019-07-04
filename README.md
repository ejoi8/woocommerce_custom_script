# Woocommerce custom script

# Adding the new custom field on the WooCommerce checkout page (date)
        // Adding the new custom field on the WooCommerce checkout page
        add_action( 'woocommerce_after_checkout_billing_form', 'display_extra_fields_after_billing_address' , 10, 1 );
        function display_extra_fields_after_billing_address () {
          _e( "<label for='delivery_date' style='font-weight:bold;'>Delivery Date:</label> ", "add_extra_fields");
          ?>
          <br>
          <input type="text" name="add_delivery_date" class="add_delivery_date" placeholder="Delivery Date">
          <script>
              jQuery(document).ready(function( $ ) {
                  $( ".add_delivery_date").datepicker( {
                    minDate: 0,	
                  } );
               } );
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

        // add order delivery date to order
        add_action( 'woocommerce_checkout_update_order_meta', 'add_order_delivery_date_to_order' , 10, 1);
        function add_order_delivery_date_to_order ( $order_id ) {
          if ( isset( $_POST ['add_delivery_date'] ) &&  '' != $_POST ['add_delivery_date'] ) {
            add_post_meta( $order_id, '_delivery_date',  sanitize_text_field( $_POST ['add_delivery_date'] ) );
          }
        }

        // add delivery date to emails
        add_filter( 'woocommerce_email_order_meta_fields', 'add_delivery_date_to_emails' , 10, 3 );
        function add_delivery_date_to_emails ( $fields, $sent_to_admin, $order ) {
            if( version_compare( get_option( 'woocommerce_version' ), '3.0.0', ">=" ) ) {            
                $order_id = $order->get_id();
            } else {
                $order_id = $order->id;
            }
            $delivery_date = get_post_meta( $order_id, '_delivery_date', true );
            if ( '' != $delivery_date ) {
          $fields[ 'Delivery Date' ] = array(
              'label' => __( 'Delivery Date', 'add_extra_fields' ),
              'value' => $delivery_date,
          );
            }
            return $fields;
        }
