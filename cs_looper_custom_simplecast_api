<?php 

//Create Simplecast API Looper Function with Args
add_filter( 'cs_looper_custom_simplecast_api', function( $result, $args ) {

  $transient_key = 'my_prefix_' . md5(serialize($args)); //Added
  $cached_result = get_transient($transient_key); //Added

  if ($cached_result !== false) { //Added
    return $cached_result; //Added
  } else { //Added

  $api_key        = isset( $args['api_key']) ? $args['api_key'] : null;
  $url            = isset( $args['url']) ? $args['url'] : null;
  $endpoint       = isset( $args['endpoint']) ? $args['endpoint'] : null;
  $path           = !empty( $args['path']) ? $args['path'] : null;
  $podcast        = !empty( $args['podcast']) ? "podcast=" . $args['podcast'] : null;
  $episode        = !empty( $args['episode']) ? "episode=" . $args['episode'] : null;
  $interval       = !empty( $args['interval']) ? "&interval=" . $args['interval'] : null;
  $sort           = !empty( $args['sort']) ? "&sort=" . $args['sort'] : null;
  $start_date     = !empty( $args['start_date']) ? "&start_date=" . $args['start_date'] : null;
  $end_date       = !empty( $args['end_date']) ? "&end_date=" . $args['end_date'] : null;
  $intervalForIf  = !empty( $args['interval']) ? $args['interval'] : null; 
  $sortForIf      = !empty( $args['sort']) ? $args['sort'] : null;

  // This assembles the endpoint and the parameters to form a working url.
  
  if ($intervalForIf == 'year') {
    // This assures that the Yearly Interval parameter returns monthly results, which will be summarized later on.
    $interval = '&interval=month';
  } 

  if ( !empty($podcast) ) {
    $assembled_url = $endpoint . "?" . $podcast . $interval . $sort . $start_date;
  } elseif ( !empty($episode)) {
    $assembled_url = $endpoint . "?" . $episode . $interval . $sort . $start_date;
  }

  // This function requires a helper script: 
  // debug_to_console($assembled_url);

  if ( !$url ) {
    $url = $assembled_url;
  }

  if ( $url ) {

    $curl = curl_init($url);
    curl_setopt($curl, CURLOPT_URL, $url);
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);

    $headers = array(
      "Accept: application/json",
      "Authorization: Bearer " . $api_key,
    );
    curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
    //for debug only!
    curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false);
    curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);

    $request = curl_exec($curl);
    curl_close($curl);
    $response = json_decode( $request, true);

    //Playing with dates to get published date and days since published

    //If this API call has an episode number
    if ( isset( $episode ) ) {

      //Get published Date from JSON Array
      $publishDateString = $response['feeds']['collection'][0]['published_at'];

      //Convert to PHP DateTime
      $publishDate = new DateTime($publishDateString);

      //Get today's Date
      $today = new DateTime();

      //Get difference between $publishDate and $today
      $timeSince = $today->diff($publishDate);

      //Convert $timeSince to Days
      $daysSince = $timeSince->days;

      //Assign proper $interval based on publish date
      if ( $daysSince < 30 ) {
        $newInterval = "&interval=day";
        // debug_to_console("Day");
      } elseif ($daysSince < 90 ) {
        $newInterval = "&interval=week";
        // debug_to_console("Week");
      } else {
        $newInterval = "&interval=month";
        // debug_to_console("Month");
      }

      if ( $intervalForIf == 'smart' ) {
        // debug_to_console($intervalForIf);
      $url = $endpoint . "?" . $episode . $newInterval . $sort;


      $curl = curl_init($url);
      curl_setopt($curl, CURLOPT_URL, $url);
      curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);

      $headers = array(
        "Accept: application/json",
        "Authorization: Bearer " . $api_key,
      );
      curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
      //for debug only!
      curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false);
      curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);

      $request = curl_exec($curl);
      curl_close($curl);
      $response = json_decode( $request, true);
      }
    }

    if ( $intervalForIf == 'year' ) {
      $filteredData = $response['by_interval'];

      $summaryByYear = [];

      foreach ($filteredData as $entry) {
        // Extract the year from the date (e.g., "2021-01" becomes "2021")
        $year = explode('-', $entry['interval'])[0];

        // Search for an existing year entry in $summaryByYear
        $found = false;
        foreach ($summaryByYear as &$yearSummary) {
          if ($yearSummary['interval'] === $year) {
            $yearSummary['downloads_total'] += $entry['downloads_total'];
            $found = true;
            break;
          }
        }
        unset($yearSummary); // Unset reference to last element
    
        // If year was not found, add a new year entry
        if (!$found) {
          $summaryByYear[] = ['interval' => $year, 'downloads_total' => $entry['downloads_total']];
        }
      }
      $path = null;
      $response = $summaryByYear;
    } 

    // debug_to_console($intervalForIf);

    //Assign path specified by parameters
    $result = !$path ? $response : $response[$path];

    // Attempting to reverse the order of the array because "&sort=" doesn't appear to be working at the API level.
    if( $sortForIf == "desc" ) {
        $result = array_reverse($result);
        }
        
    }
    //Added
    set_transient($transient_key, $result, 15 * MINUTE_IN_SECONDS); //Added
    return $result;
  } //Added
}, 10, 2);
