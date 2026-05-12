<?php
/**
 * Plugin Name: Auto Product Creator From Media
 * Description: Creates a new simple product in draft status when a media file is uploaded.
 * Version: 1.0
 * Author: Israel Mateo Manzano
 * Author URI: https://cachitoswp.com
 */

if (!defined('ABSPATH')) {
    exit; // Exit if accessed directly
}

function apc_enqueue_admin_styles($hook) {
    if ('toplevel_page_auto_product_creator' !== $hook) {
        return;
    }
    wp_enqueue_style('apc-admin-styles', plugin_dir_url(__FILE__) . 'apcm-admin-styles.css', array(), '1.0.0');
}
add_action('admin_enqueue_scripts', 'apc_enqueue_admin_styles');


// Add settings link on plugin page
add_filter('plugin_action_links_' . plugin_basename(__FILE__), 'apc_add_settings_link');

function apc_add_settings_link($links) {
    $settings_link = '<a href="options-general.php?page=auto_product_creator">' . __('Settings') . '</a>';
    array_unshift($links, $settings_link);
    return $links;
}

// Add this near the top of the file, after the initial plugin header

// Display admin notice when plugin is enabled
add_action('admin_notices', 'apc_admin_notice');

function apc_admin_notice() {
    $options = get_option('apc_settings');
    if (isset($options['apc_enable_plugin']) && $options['apc_enable_plugin']) {
        echo '<div class="notice notice-warning is-dismissible">
            <p><strong>Auto Product Creator:</strong> The automatic product creation feature is currently enabled. New products will be created for each media upload until deactivated.</p>
        </div>';
    }
}


// Add settings page
add_action('admin_menu', 'apc_add_admin_menu');
add_action('admin_init', 'apc_settings_init');

function apc_add_admin_menu() {
    add_menu_page(
        'Auto Product Creator',
        'Auto Product Creator',
        'manage_options',
        'auto_product_creator',
        'apc_options_page',
        'dashicons-products',
        56
    );
}

function apc_settings_init() {
    
    register_setting('apc_plugin_page', 'apc_settings');

    add_settings_section(
        'apc_plugin_page_section',
        __('Plugin Settings', 'auto-product-creator-from-media'),
        'apc_settings_section_callback',
        'apc_plugin_page'
    );

    add_settings_field(
        'apc_enable_plugin',
        __('Enable Auto Product Creation', 'auto-product-creator-from-media'),
        'apc_enable_plugin_render',
        'apc_plugin_page',
        'apc_plugin_page_section'
    );

    add_settings_field(
        'apc_default_category',
        __('Default Product Category', 'auto-product-creator-from-media'),
        'apc_default_category_render',
        'apc_plugin_page',
        'apc_plugin_page_section'
    );

    add_settings_field(
        'apc_default_tags',
        __('Default Product Tags', 'auto-product-creator-from-media'),
        'apc_default_tags_render',
        'apc_plugin_page',
        'apc_plugin_page_section'
    );

    add_settings_field(
        'apc_default_price',
        __('Default Product Price', 'auto-product-creator-from-media'),
        'apc_default_price_render',
        'apc_plugin_page',
        'apc_plugin_page_section'
    );

    add_settings_field(
        'apc_default_weight',
        __('Default Product Weight', 'auto-product-creator-from-media'),
        'apc_default_weight_render',
        'apc_plugin_page',
        'apc_plugin_page_section'
    );

    add_settings_field(
        'apc_default_dimensions',
        __('Default Product Dimensions', 'auto-product-creator-from-media'),
        'apc_default_dimensions_render',
        'apc_plugin_page',
        'apc_plugin_page_section'
    );

    add_settings_field(
        'apc_custom_taxonomies',
        __('Custom Taxonomies', 'auto-product-creator-from-media'),
        'apc_custom_taxonomies_render',
        'apc_plugin_page',
        'apc_plugin_page_section'
    );
}

function apc_settings_section_callback() {
    echo __('Enable or disable automatic product creation on media upload.', 'auto-product-creator-from-media');
}

function apc_enable_plugin_render() {
    $options = get_option('apc_settings');
    ?>
    <input id="apc_enable_plugin" type='checkbox' name='apc_settings[apc_enable_plugin]' <?php checked(isset($options['apc_enable_plugin']), 1); ?> value='1'>
    <label for="apc_enable_plugin" class="checkbox-toggle"></label> <span></span>
    <?php
}

function apc_options_page() {
    ?>
    <div class="wrap apc-settings-page">
        <figure>
            <img src="<?php echo plugin_dir_url(__FILE__).'/splash-autocreate-products-media-uploads.webp';?>" alt="Auto Product Creator Settings">
        </figure>
        <div class="apc-settings-page--inner">
            <h1>Auto Product Creator Settings</h1>
            <form action='options.php' method='post'>
                <?php
                settings_fields('apc_plugin_page');
                do_settings_sections('apc_plugin_page');
                submit_button('Save Settings', 'apc-button');
                ?>
            </form>
            <footer class="apc-footer">
                <p>Created by <a href="https://cachitoswp.com" target="_blank" title="CachitosWP WordPress Development Blog and Services by Israel Mateo">cachitoswp.com</a> follow up for more!</p>
            </footer>
        </div>
    </div>
    <?php
}

// Main functionality
add_action('add_attachment', 'apc_create_simple_product_automatically', 9999);

function apc_default_category_render() {
    $options = get_option('apc_settings', array());
    $categories = get_terms(['taxonomy' => 'product_cat', 'hide_empty' => false]);
    ?>
    <select name='apc_settings[apc_default_category]'>
        <option value=''>Select a category</option>
        <?php foreach ($categories as $category) : ?>
            <option value='<?php echo $category->term_id; ?>' <?php selected(isset($options['apc_default_category']) ? $options['apc_default_category'] : '', $category->term_id); ?>>
                <?php echo $category->name; ?>
            </option>
        <?php endforeach; ?>
    </select>
    <?php
}


function apc_default_tags_render() {
    $options = get_option('apc_settings', array());
    ?>
    <input type='text' name='apc_settings[apc_default_tags]' value='<?php echo $options['apc_default_tags'] ?? ''; ?>'>
    <p class="description">Enter tags separated by commas</p>
    <?php
}

function apc_default_price_render() {
    $options = get_option('apc_settings', '');
    ?>
    <input type='number' step='0.01' name='apc_settings[apc_default_price]' value='<?php echo $options['apc_default_price'] ?? ''; ?>'>
    <?php
}

function apc_default_weight_render() {
    $options = get_option('apc_settings');
    ?>
    <input type='text' name='apc_settings[apc_default_weight]' value='<?php echo $options['apc_default_weight'] ?? ''; ?>' placeholder='Weight'>
    <?php
}

function apc_default_dimensions_render() {
    $options = get_option('apc_settings', array());
    ?>
    <input type='text' name='apc_settings[apc_default_length]' value='<?php echo $options['apc_default_length'] ?? ''; ?>' placeholder='Length'>
    <input type='text' name='apc_settings[apc_default_width]' value='<?php echo $options['apc_default_width'] ?? ''; ?>' placeholder='Width'>
    <input type='text' name='apc_settings[apc_default_height]' value='<?php echo $options['apc_default_height'] ?? ''; ?>' placeholder='Height'>
    <?php
}



function apc_custom_taxonomies_render() {
    $options = get_option('apc_settings', array());
    $custom_taxonomies = isset($options['apc_custom_taxonomies']) ? $options['apc_custom_taxonomies'] : array(array('taxonomy' => '', 'terms' => ''));
    ?>
    <div id="apc-custom-taxonomies">
        <?php foreach ($custom_taxonomies as $index => $taxonomy) : ?>
            <div class="apc-taxonomy-row">
                <input type="text" name="apc_settings[apc_custom_taxonomies][<?php echo $index; ?>][taxonomy]" value="<?php echo esc_attr($taxonomy['taxonomy']); ?>" placeholder="Taxonomy">
                <input type="text" name="apc_settings[apc_custom_taxonomies][<?php echo $index; ?>][terms]" value="<?php echo esc_attr($taxonomy['terms']); ?>" placeholder="Comma separated term IDs">
                <button type="button" class="apc-button apc-remove-taxonomy">Remove</button>
            </div>
        <?php endforeach; ?>
    </div>
    <button type="button" class="apc-button apc-add-taxonomy">Add Taxonomy</button>

    <script>
    jQuery(document).ready(function($) {
        var index = <?php echo count($custom_taxonomies); ?>;
        $('.apc-add-taxonomy').on('click', function() {
            var newRow = '<div class="apc-taxonomy-row">' +
                '<input type="text" name="apc_settings[apc_custom_taxonomies][' + index + '][taxonomy]" placeholder="Taxonomy">' +
                '<input type="text" name="apc_settings[apc_custom_taxonomies][' + index + '][terms]" placeholder="Terms (comma-separated)">' +
                '<button type="button" class="apc-button apc-remove-taxonomy">Remove</button>' +
                '</div>';
            $('#apc-custom-taxonomies').append(newRow);
            index++;
        });

        $(document).on('click', '.apc-remove-taxonomy', function() {
            $(this).closest('.apc-taxonomy-row').remove();
        });
    });
    </script>
    <?php
}




function apc_create_simple_product_automatically($image_id) {
    $options = get_option('apc_settings');
    if (!isset($options['apc_enable_plugin']) || !$options['apc_enable_plugin']) {
        return;
    }

    if (!class_exists('WC_Product_Simple')) {
        return;
    }

    $product = new WC_Product_Simple();
    
    // Set product name from image title
    $product->set_name(get_the_title($image_id));
    
    // Set product status to draft
    $product->set_status('draft');
    
    $product->set_catalog_visibility('visible');
    
    // Set default category
    if (!empty($options['apc_default_category'])) {
        $product->set_category_ids(array($options['apc_default_category']));
    }
    
    // Set default tags
    if (!empty($options['apc_default_tags'])) {
        $tags = explode(',', $options['apc_default_tags']);
        $product->set_tag_ids($tags);
    }
    
    // Set default price
    if (!empty($options['apc_default_price'])) {
        $product->set_regular_price($options['apc_default_price']);
    }

    // Set default weight
if (!empty($options['apc_default_weight'])) {
    $product->set_weight($options['apc_default_weight']);
}
    
    // Set default dimensions
    if (!empty($options['apc_default_length']) && !empty($options['apc_default_width']) && !empty($options['apc_default_height'])) {
        $product->set_length($options['apc_default_length']);
        $product->set_width($options['apc_default_width']);
        $product->set_height($options['apc_default_height']);
    }

    // Set custom taxonomies
    if (!empty($options['apc_custom_taxonomies'])) {
        foreach ($options['apc_custom_taxonomies'] as $custom_taxonomy) {
            if (!empty($custom_taxonomy['taxonomy']) && !empty($custom_taxonomy['terms'])) {
                $terms = array_map('trim', explode(',', $custom_taxonomy['terms']));
                wp_set_object_terms($product->get_id(), (int)$terms, $custom_taxonomy['taxonomy']);
            }
        }
    }
    
    // Set product description from image caption
    $attachment = get_post($image_id);
    $product->set_description($attachment->post_content);
    
    // Set product short description from image alt text
    $product->set_short_description(get_post_meta($image_id, '_wp_attachment_image_alt', true));
    
    $product->set_image_id($image_id);
    
    // Save the product
    $product->save();
}

