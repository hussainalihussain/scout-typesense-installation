# How to use Scout Typesense in Windows OS

`Scout` is the laravel package that helps create server for `Full Text Searching` and `Typesense` is the Open Source Engine, fast, support keyword search, semantic search, geo search, and vector search.

The below are the steps to install and use `Typesense` (using Scout) in windows Operating System. The hardest part in this is the installation of `Typesense` and may the configuration also a little harder if you missed up something.

## Step#1 - Install Scout Package
Goto the official document of laravel of [Scout Package](https://laravel.com/docs/10.x/scout)

Install scout:
> composer require laravel/scout

## Step#2 - Publish config file
Publish config i.e. config/scout.php file by using:
> php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

## Step#3 - Use Searchable in Model
Example:
```
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;
}
```

## Step#4 - Downloading and installing Typesense
Typesense as a `Self Hosting` (Hosted) in windows is a little harder.


### Step-I - Install WSL/Linux
See the [full documentation](https://learn.microsoft.com/en-us/windows/wsl/install).

#### 1
Type the following to see the list of available Linux distributions.

> wsl --install

#### 2
Now Install one of you favorite distribution. For example if i want to install `Ubuntu-20.04` then I need:

> wsl --install -d Ubuntu-20.04

*Note:*
This may ask to setup (root) user with password.

#### 3
After installation you can check which version of Linux Distro. is running by:
> wsl -l -v

#### 4 - Open Linux
Just type `Ubuntu` in the Search of Start Menu and you will see (in my case) `Ubuntu 20.04.6 LTS`, just open it and this will open `powershell`

## Step#5 - Download Typesense
After starting `Ubuntu` (in Powershell) now type the following commands:
> curl -O https://dl.typesense.org/releases/0.25.2/typesense-server-0.25.2-amd64.deb

> sudo apt install ./typesense-server-0.25.2-amd64.deb

## Step#6 - Start Typesense
Type the following command to start `Typesense` in your (WSL) `Ubuntu`.
>sudo /usr/bin/./typesense-server --config=/etc/typesense/typesense-server.ini

## Step#7 - Test Typesense
Just Type (in your `Windows` Terminal Or in your `Ubuntu` Terminal):
> curl http://localhost:8108/health

and if you see the following message, this means your `typesense` is runnig.
```
{"ok":true}
```

## Step#8 - Configuration of Typesense Server
By default, Typesense will start on port `8108`, and the installation will generate a random API key, which you can view/change from the configuration file at `/etc/typesense/typesense-server.ini` to view the file just do:
> cat /etc/typesense/typesense-server.ini

and you will see the whole settings, so copy everything specially api-key

## Step#9 - Install SDK (Package) for Typesense in Laravel
Just run:
> composer require typesense/typesense-php 

## Step#10 - Update .env:
```
SCOUT_DRIVER=typesense
TYPESENSE_API_KEY=masterKey
TYPESENSE_HOST=localhost
```
Paste the `api-key` you have copied in the Step#8

If you have changed the default setting then you can add the follwing to the `.env`:
```
TYPESENSE_PORT=8108
TYPESENSE_PATH=
TYPESENSE_PROTOCOL=http
```

## Step#11 - Prepare Data for Storage in Typesense
Add something like below to your laravel `Model` i.e. fields that you need to make searchable:
```
public function toSearchableArray()
{
    return array_merge($this->toArray(),[
        'id' => (string) $this->id,
        'created_at' => $this->created_at->timestamp,
    ]);
}
```

## Step#12 - For Soft deletable Model
If your searchable model is soft `deletable`, you should define a __soft_deleted field in the model's corresponding Typesense schema within your application's config/scout.php configuration file:
```
User::class => [
    'collection-schema' => [
        'fields' => [
            // ...
            [
                'name' => '__soft_deleted',
                'type' => 'int32',
                'optional' => true,
            ],
        ],
    ],
],
```
In my example where I used the setting was:
```
    'typesense' => [
        'client-settings' => [
            'api_key' => env('TYPESENSE_API_KEY', 'xyz'),
            'nodes' => [
                [
                    'host' => env('TYPESENSE_HOST', 'localhost'),
                    'port' => env('TYPESENSE_PORT', '8108'),
                    'path' => env('TYPESENSE_PATH', ''),
                    'protocol' => env('TYPESENSE_PROTOCOL', 'http'),
                ],
            ],
            'nearest_node' => [
                'host' => env('TYPESENSE_HOST', 'localhost'),
                'port' => env('TYPESENSE_PORT', '8108'),
                'path' => env('TYPESENSE_PATH', ''),
                'protocol' => env('TYPESENSE_PROTOCOL', 'http'),
            ],
            'connection_timeout_seconds' => env('TYPESENSE_CONNECTION_TIMEOUT_SECONDS', 2),
            'healthcheck_interval_seconds' => env('TYPESENSE_HEALTHCHECK_INTERVAL_SECONDS', 30),
            'num_retries' => env('TYPESENSE_NUM_RETRIES', 3),
            'retry_interval_seconds' => env('TYPESENSE_RETRY_INTERVAL_SECONDS', 1),
        ],
        'model-settings' => [
            Post::class => [
                'collection-schema' => [
                    'fields' => [
                        [
                            'name' => 'id',
                            'type' => 'string',
                        ],
                        [
                            'name' => 'title',
                            'type' => 'string',
                            // 'optional' => false,
                        ],
                    ],
                    'default_sorting_field' => 'created_at',
                ],
                'search-parameters' => [
                    'query_by' => 'title'
                ],
            ],
        ],
    ],
```

**Note:** The other setting was already present in the `config/scout.php` file, I just added the following:
```
Post::class => [
    'collection-schema' => [
        'fields' => [
            [
                'name' => 'id',
                'type' => 'string',
            ],
            [
                'name' => 'title',
                'type' => 'string',
            ],
        ],
        'default_sorting_field' => 'created_at',
    ],
    'search-parameters' => [
        'query_by' => 'title'
    ],
],
```

## Step#13 - Test Typesense Searching
In my case I am testing `Post` Model, see the following example:

```
$text  = 'post day 4 morning';
$posts = Post::search($text)->get();

dd($posts->toArray());
```

