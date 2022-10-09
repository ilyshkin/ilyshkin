<?php

const MP3_DIR = '/drive2/Dropbox/backup/mp3/';

require_once __DIR__ . '/vendor/autoload.php';

$curl = new \Zelenin\Curl();

$playlist_url = 'https://music.yandex.ru/users/k4lashnikov.ilja/playlists/3';

preg_match_all( '/users\/(.*)\/playlists\/(.*)/isu', $playlist_url, $matches );

$owner = $matches[1][0];
$playlist_id = $matches[2][0];

$response = $curl->get( 'http://music.yandex.ru/get/playlist2.xml?kinds=' . $playlist_id . '&owner=' . $owner );

$playlist = json_decode( $response['body'], true );

$playlist_title = $playlist['playlists'][0]['title'];

$tracks = implode( ',', $playlist['playlists'][0]['tracks'] );

$response = $curl->get( 'http://music.yandex.ru/get/tracks.xml?tracks=' . urlencode( $tracks ) );

$tracks = json_decode( $response['body'], true );
$tracks = $tracks['tracks'];

$playlist_dir = MP3_DIR . $playlist_title;
if ( !file_exists( $playlist_dir ) && !is_dir( $playlist_dir ) ) {
  mkdir( $playlist_dir );
}

foreach ( $tracks as $track ) {
  $artist =  $track['artist'];
  $title = $track['title'];

  $response = $curl->get( 'http://storage.music.yandex.ru/download-info/' . $track['storage_dir'] . '/2.mp3' );

  $xml = new DOMDocument();
  $xml->loadXML( $response['body'] );

  $host = $xml->getElementsByTagName( 'host' )->item(0)->nodeValue;
  $ts = $xml->getElementsByTagName( 'ts' )->item(0)->nodeValue;
  $path = $xml->getElementsByTagName( 'path' )->item(0)->nodeValue;
  $s = $xml->getElementsByTagName( 's' )->item(0)->nodeValue;
  $n = md5( 'XGRlBW9FXlekgbPrRHuSiA' . substr( $path, 1 ) . $s );

  $mp3_url = 'http://' . $host . '/get-mp3/' . $n . '/' . $ts . $path;
  
  //echo $mp3_url . PHP_EOL;

  $response = $curl->get( $mp3_url );
  $mp3_name = addslashes( $artist . ' - ' . $title . '.mp3' );
  echo $mp3_name . PHP_EOL;
  file_put_contents( MP3_DIR . $playlist_title . '/' . $mp3_name, $response['body'] );
}
