# Flysystem Adapter for Google Drive

[![Author](https://img.shields.io/badge/author-nao--pon%20hypweb-blue.svg?style=flat)](http://xoops.hypweb.net/)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)


## Installation

- For Google Drive API V3
```bash
composer require modernmcguire/flysystem-google-drive:~1.1
```
- For Google Drive API V2 "**Deprecated**"
```bash
composer require modernmcguire/flysystem-google-drive:~1.0.0
```

#### Follow [Google Docs](https://developers.google.com/drive/v3/web/enable-sdk) to obtain your `ClientId, ClientSecret & refreshToken` in addition you can also check this [easy-to-follow tutorial](https://gist.github.com/ivanvermeyen/cc7c59c185daad9d4e7cb8c661d7b89b)


1. Add a `google` disk to your `config/filesystems.php`.

```php
'disks' => [

	/* ... */

	'google' => [
		'driver' => 'google',
		'clientId' => env('GOOGLE_DRIVE_CLIENT_ID'),
		'clientSecret' => env('GOOGLE_DRIVE_CLIENT_SECRET'),
		'refreshToken' => env('GOOGLE_DRIVE_REFRESH_TOKEN'),
		'folderId' => env('GOOGLE_DRIVE_FOLDER_ID'),
		'teamDriveId' => env('GOOGLE_DRIVE_TEAM_DRIVE_ID'),
	],
]
```

2. Create a new Service Provider (`App\Providers\GoogleDriveServiceProvider.php`) to contain all of the setup logic.

```php
<?php

namespace App\Providers;

use Google_Client;
use Google_Service_Drive;
use League\Flysystem\Filesystem;
use App\Http\Livewire\GoogleDriveUI;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use Hypweb\Flysystem\GoogleDrive\GoogleDriveAdapter;

class GoogleDriveServiceProvider extends ServiceProvider
{
	public function register()
	{
		//
	}

	public function boot()
	{
		Storage::extend('google', function ($app, $config) {
			$client = new Google_Client();
			$client->setClientId($config['clientId']);
			$client->setClientSecret($config['clientSecret']);
			$client->refreshToken($config['refreshToken']);
			$service = new Google_Service_Drive($client);

			$options = [];
			if (isset($config['teamDriveId'])) {
				$options['teamDriveId'] = $config['teamDriveId'];
			}

			$options['additionalFetchField'] = 'thumbnailLink,contentHints,webContentLink,webViewLink,iconLink';

			$adapter = new GoogleDriveAdapter($service, $config['folderId'], $options);

			if (isset($config['teamDriveId']) && ! empty($config['teamDriveId'])) {
				// Reset the pathPrefix and root back to your custom parent folder
				$adapter->setPathPrefix($config['folderId']);
				$adapter->root = $config['folderId'];
			}

			// if we want to implement Flysystem caching
			// $cacheStore = new \League\Flysystem\Cached\Storage\Memory();
			// $adapter = new \League\Flysystem\Cached\CachedAdapter($adapter, $cacheStore);
			// return new Filesystem($adapter);

			return new Filesystem($adapter);
		});
	}
}
```

3. Add the service provider to your providers array in `config/app.php`

```php
'providers' => [

	/* ... */

	/*
     * Application Service Providers...
     */
	App\Providers\AppServiceProvider::class,
	App\Providers\AuthServiceProvider::class,
	App\Providers\BladeServiceProvider::class,
	// App\Providers\BroadcastServiceProvider::class,
	App\Providers\EventServiceProvider::class,

	App\Providers\GoogleDriveServiceProvider::class,


]
```

## Usage
You can also check [This Example](https://github.com/modernmcguire/flysystem-google-drive/blob/master/example/GoogleUpload.php) for a better understanding of how to use the library directly.

You can see below for the indirect usage utilizing the Laravel Filesystem.

```php
// get folders and files beneath a parent folder
Storage::disk('google')->listContents('/', $recursive = true);

Storage::disk('google')->get($path);
Storage::disk('google')->put($path, 'Some contents');

Storage::disk('google')->download($path, $filename);

Storage::disk('google')->delete($path);
Storage::disk('google')->deleteDirectory($path);

Storage::disk('google')->makeDirectory($path . '/Some new folder');

Storage::disk('google')->copy($source, $destination);
Storage::disk('google')->move($source, $destination);

Storage::disk('google')->setVisibility($path, 'public');
Storage::disk('google')->setVisibility($path, 'private');
```

## Tips

- [Setup a Laravel Storage driver with Google Drive API](https://gist.github.com/ivanvermeyen/cc7c59c185daad9d4e7cb8c661d7b89b)
- [Issue list with "HowTo" tag](https://github.com/modernmcguire/flysystem-google-drive/issues?utf8=%E2%9C%93&q=label%3AHowTo%20)
