# Laravel Console Command - All Features

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use DB;

class OnexSystemReset extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'onex:system-reset';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'System Database Reset - For New Installation';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        try {

            $isConfirm = $this->ask("Are you sure, You want to reset the system? (Y/N)");
            if (strtolower($isConfirm) == "y") {
                $this->comment('System is resetting..., Please wait...');

                $count = 20;
                $this->output->progressStart($count);

                DB::table('users')->where('id', '!=', 1)->delete();
                DB::table('user_roles')->where('user_id', '!=', 1)->delete();
                DB::table('users_profile')->truncate();
                
                DB::table('product_subcategory')->truncate();
                DB::table('product_category')->truncate();

                DB::table('product_variants')->truncate();
                DB::table('product_master')->truncate();
                DB::table('product_meta_fields')->truncate();
                DB::table('product_images')->truncate();
                DB::table('product_bundle_free')->truncate();
                
                DB::table('unit_master')->truncate();
                DB::table('brands')->truncate();
                
                DB::table('batch_products')->truncate();
                DB::table('batches')->truncate();

                DB::table('sale_products')->truncate();
                DB::table('sales')->truncate();
                
                DB::table('purchase_products')->truncate();
                DB::table('purchase')->truncate();

                DB::table('company_information')->truncate();

                for ($i = 1; $i <= 20; $i++) {
                    sleep(1);
                    $this->output->progressAdvance();
                }

                $this->output->progressFinish();

                $this->info('System has been reset successfully!');
            } else {
                $this->line('Cancelled!!');
            }
        
        } catch (\Exception $e) {
            $this->error('Query execution failed');
            $this->comment($e->getMessage());
            //return $e->getMessage();
        }
    }
}
```

## Arguments - Options

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Str;
use Hash;
use Helper;
use DB;

class OnexUser extends Command
{
    /**
     * The name and signature of the console command.
     * 
     * @var string
     */
    protected $signature = 'onex:set-user {userType?} {--email=} {--password=} {--username=} {--entry=1} {--role=1}';

    // onex:set-user {userType=client} {--email=} {--password=}

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Command description';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        try {

            $argUserType = $this->argument('userType');
            $optEmail = $this->option('email');
            $optPassword = $this->option('password');
            $optUsername = $this->option('username');
            $optEntry = $this->option('entry');
            $optRole = $this->option('role');

            if (!empty($argUserType) && strtolower($argUserType) == "dev") {
                
                $this->comment('Dev user setting up...');
                
                $email = !empty($optEmail) ? $optEmail : 'devteam@onexcrm.com';
                $password = !empty($optPassword) ? $optPassword : 'Onex#123456';
                $username = !empty($optUsername) ? $optUsername : 'onexdev';
                
                DB::table('users')->where('id', 1)->delete();
                DB::table('users')->insert([
                    'id' => 1,
                    'hash_id' => Str::uuid(36)->toString(),
                    'unique_id' => Helper::userUniqueId(),
                    'first_name' => 'Onex',
                    'last_name' => 'Dev',
                    'email_id' => $email,
                    'password' => Hash::make($password),
                    'user_name' => $username,
                    'phone_number' => '9836395513',
                    'whatsapp_number' => '9836395513',
                    'gender' => 'Male',
                    'is_crm_access' => 1,
                    'status' => 1,
                    'user_category' => 1
                ]);
                DB::table('user_roles')->where('user_id', 1)->delete();
                DB::table('user_roles')->insert(['user_id' => 1, 'role_id' => 1]);

                $this->info('Dev user setup done!');
                $this->line('Email = ' . $email);
                $this->line('Username = ' . $username);
                $this->line('Password = ' . $password);
                $this->info('Lets login to CRM !!');
            }

            if (empty($argUserType) || (!empty($argUserType) && strtolower($argUserType) == "client")) {
                
                $this->comment('Client user setting up...');
                
                $email = !empty($optEmail) ? $optEmail : 'testclient' . rand(123, 999) . '@onexcrm.com';
                $password = !empty($optPassword) ? $optPassword : 'Test#123456';
                $username = !empty($optUsername) ? $optUsername : 'democlient' . rand(123, 999);
                $entry = !empty($optEntry) ? $optEntry : 1;
                $role = !empty($optRole) ? $optRole : 1;
                
                for ($i = 1; $i <= $entry; $i++) {
                    $clientId = DB::table('users')->insertGetId([
                        'hash_id' => Str::uuid(36)->toString(),
                        'unique_id' => Helper::userUniqueId(),
                        'first_name' => 'Test',
                        'last_name' => 'Client',
                        'email_id' => $email,
                        'password' => Hash::make($password),
                        'user_name' => $username,
                        'is_crm_access' => 1,
                        'status' => 1,
                        'user_category' => 1
                    ]);
                    DB::table('user_roles')->insert(['user_id' => $clientId, 'role_id' => $role]);

                    if ($i == 1) {
                        $this->info('Client user setup done!');
                    }
                    $this->line('Email = ' . $email);
                    $this->line('Username = ' . $username);
                    $this->line('Password = ' . $password);
                    if ($i > 1) {
                        $this->line("-----------------------------------------------------");
                    }
                    if ($i == $entry) {
                        $this->info('Lets login to CRM !!');
                    }
                }
            }
        
        } catch (\Exception $e) {
            $this->error('Query execution failed');
            $this->comment($e->getMessage());
            //return $e->getMessage();
        }
    }
}

```
