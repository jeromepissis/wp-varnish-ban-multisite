<?php
/**
 * Plugin Name: Varnish purge multi site
 * Plugin URI:
 * Version: 1.0
 * Author: Jérôme Pissis <jerome.pissis@gmail.com>
 * Description: Purge varnish cache when a post was publish
 */

class VarnishBanMultiSite
{
    private $site_path         = '';
    private $site_domain       = '';
    private $real_url          = '';
    private $varnish_server    = '';

    public function __construct()
    {
        add_action('pre_post_update' , array($this, 'vbms_before_save_post') , 99);
        add_action('transition_post_status', array($this, 'vbms_post_transitionStatus'), 99);

        global $current_blog;

        $this->site_path       = substr($current_blog->path, 0, -1);
        $this->site_domain     = $current_blog->domain;

        // Please add your ip server varnish and set the port.
        $this->varnish_server  = array('IP SERVER A:PORT', 'IP SERVER B:PORT');
    }

    /**
     * Save real url before save post
     */
    function vbms_before_save_post() 
    {
        global $post;

        $this->real_url = get_permalink($post->ID);
    }

    /**
     * For each change of status post
     */
    function vbms_post_transitionStatus( $new_status, $old_status = '', $post = '') 
    {
        global $post;

        if ( is_admin() ) {            
            if ($new_status == 'publish'){
                $url = get_permalink($post->ID);
            } else{
                $url = $this->real_url;
            } 

            // New publication 
            if ($new_status == 'publish' && $post->post_status != 'publish'){
                $this->vbms_PurgeHomePage();
                $this->vbms_PurgePageRef();   
            } 

            // If an existing article was modify
            else if ($new_status == 'publish' && $post->post_status == 'publish'){
                $this->vbms_PurgeHomePage();
                $this->vbms_PurgePageRef();
                $this->vbms_purgeSingle($url);
            } 

            // Set an article as draft
            else if ($new_status == 'draft' && $post->post_status == 'publish'){
                $this->vbms_PurgeHomePage();
                $this->vbms_PurgePageRef();
            }  
        } 
    }

    /**
     * Purge an url
     */
    public function vbms_PurgeSingle($url)
    {
        $url_element = str_replace(get_bloginfo('url'), '', $url);

        if ($url_element != '' && $url_element != '/'){
            $this->vbms_purgeObjectByReqUrl($this->site_path.$url_element);
        }
    }

    /**
     * Purge the home page
     */
    public function vbms_PurgeHomePage()
    {
        $this->vbms_purgeObjectByReqUrl($this->site_path.'/');
    }

    /**
     * Purge each page where the post appears 
     */
    function vbms_PurgePageRef()
    {
        global $post;

        $list_category = get_the_category($post->ID);

        if (is_array($list_category)){
            foreach ($list_category as $categ){
                if ($categ->category_parent != ''){
                    $cat_parent = get_the_category_by_ID($categ->category_parent);
                    $cat_parent = get_term_by('name', $cat_parent, 'category');

                    $this->vbms_purgeObjectByReqUrl($this->site_path."/".$cat_parent->slug);
                    $this->vbms_purgeObjectByReqUrl($this->site_path."/".$cat_parent->slug."/".$categ->slug);
                } else{
                    $this->vbms_purgeObjectByReqUrl($this->site_path."/".$categ->slug);
                }
            }
        }
    }


    /**
     * Send an ban request for each varnish server
     *
     * @param $reqUrl
     */
    public function vbms_purgeObjectByReqUrl($reqUrl)
    {   
        if (is_array($this->varnish_server)){
            foreach ($this->varnish_server as $varnish){
                exec("/usr/bin/varnishadm -S /etc/varnish/secret -T ".$varnish." ban \"req.http.host == ".$this->site_domain." && req.url == ".$reqUrl."\"");
            }
        }        
    }
}

$VarnishBanMultiSite = new VarnishBanMultiSite();