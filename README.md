Wordpress-HTTPS
====================

Original code
---------------------

Plugin - http://wordpress.org/extend/plugins/wordpress-https/
Author - http://profiles.wordpress.org/Mvied/

Why we forked
---------------------

When parsing the page for URLs, when an element (img, link etc..) was found it would perform a validation curl request to ensure that it could access the item over HTTPS.
If valid, it would append this URL in the wp_option table to a safe list. If it came across this list again, it wouldn't need to 

2 reasons why we commented out this code

+ On blog postings that had a large number of elements with http, the page was literally timing out doing all the validation curl requests
+ That serialised array in wp_options overtime becames behemoth!! And in essence you will be pulling out large amounts of data per page request. Overhead+++!


### Offending Code

	public function makeUrlHttps( $string ) {
		$url = WordPressHTTPS_Url::fromString( $string ); // URL to replace HTTP URL
		if ( $url ) {
			if ( $this->isUrlLocal($url) ) {
				$url->setScheme('https');
				$url->setHost($this->getHttpsUrl()->getHost());
				$url->setPort($this->getHttpsUrl()->getPort());

				if ( $this->getSetting('ssl_host_diff') && strpos($url->getPath(), $this->getHttpsUrl()->getPath()) === false ) {
					if ( $this->getHttpUrl()->getPath() == '/' ) {
						$url->setPath(rtrim($this->getHttpsUrl()->getPath(), '/') . $url->getPath());
					} else {
							$url->setPath(str_replace($this->getHttpUrl()->getPath(), $this->getHttpsUrl()->getPath(), $url->getPath()));
				}
			}

			$string = $url->toString();
		} else {
			if ( $url->getScheme() == 'http' && @in_array($url, $this->getSetting('secure_external_urls')) == false && @in_array($url, $this->getSetting('unsecure_external_urls')) == false ) {
				$test_url = clone $url;
				$test_url->setScheme('https');
				if ( $test_url->isValid() ) {
					// Cache this URL as available over HTTPS for future reference
					$this->addSecureExternalUrl($url->toString());
				} else {
					// If not available over HTTPS, mark as an unsecure external URL
					$this->addUnsecureExternalUrl($url->toString());
				}
			}

			if ( in_array($url->toString(), $this->getSetting('secure_external_urls')) ) {
				$string = str_replace($url, str_replace('http://', 'https://', $url), $string);
			}
		}
		unset($url);
		}
		return $string;
	}