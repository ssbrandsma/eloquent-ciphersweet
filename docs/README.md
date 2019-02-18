# Eloquent-CipherSweet Adapter Documentation

## Installation

This adapter can be installed through Composer:

```sh
composer require paragonie/eloquent-ciphersweet
```

We do not support non-Composer use-cases with this adapter library.

## Configuration

Once you've installed , create a `config/ciphersweet.php` file to [specify your CipherSweet key provider](https://github.com/paragonie/ciphersweet/blob/master/docs/README.md#define-your-key-provider).

```php
<?php
use ParagonIE\CipherSweet\CipherSweet;
use ParagonIE\CipherSweet\Backend\ModernCrypto;
use ParagonIE\CipherSweet\KeyProvider\StringProvider;

$key = new StringProvider(
    new ModernCrypto,
    env('APP_CIPHERSWEET_KEY', '2307c9579923de254e663a42dd18057ff3d955fa7de8e2f6f1bd3ab1bec3bd3c')
);
return [
    'engine' => new CipherSweet($key)
];
```

Once the configuration is done, you can begin using encrypted fields in your models.

There are two ways to achieve this effect:

### EncryptedFieldModel Base Class

The easiest way to use the features of the adapter is to ensure your models extend
`EncryptedFieldModel` instead of the base `Model`.

```diff
<?php
- use Illuminate\Database\Eloquent\Model;
+ use ParagonIE\EloquentCipherSweet\EncryptedFieldModel;

- class Foo extends Model
+ class Foo extends EncryptedFieldModel
```

This automatically loads in the trait and boots it for you. If you use this in a base
class, and some of your classes that inherit that base class *don't* need encrypted fields,
you can simply leave them un-configured.

### CipherSweet Trait

If this is not tenable due to existing object inheritance requirements, you may also
simply use the `CipherSweet` trait and then add `static::bootCipherSweet()` to your
`boot()` method, like so.

```php
<?php
use Illuminate\Database\Eloquent\Model;
use ParagonIE\EloquentCipherSweet\CipherSweet;

class Blah extends Model
{
    use CipherSweet;

    /**
     * The "booting" method of the model.
     *
     * @return void
     */
    protected static function boot()
    {
        parent::boot();
        /* ...other custom code here... */
        static::bootCipherSweet();
    }    
}
```

## Defining Encrypted Fields

Override the `configureCipherSweet()` method to add columns to the bare
`EncryptedMultiRows` object.

```php
<?php
namespace YourCompany\YourApp;

use ParagonIE\CipherSweet\BlindIndex;
use ParagonIE\CipherSweet\CompoundIndex;
use ParagonIE\CipherSweet\EncryptedMultiRows;
use ParagonIE\CipherSweet\Transformation\LastFourDigits;
use ParagonIE\EloquentCipherSweet\EncryptedFieldModel;

class Blah extends EncryptedFieldModel
{
    /**
     * @param EncryptedMultiRows $multiRows
     * @return EncryptedMultiRows
     */
    public function configureCipherSweet(
        EncryptedMultiRows $multiRows
    ): EncryptedMultiRows {
        return $multiRows
            ->addTable('sql_table_name')
                ->addTextField('sql_table_name', 'column1')
                ->addBooleanField('sql_table_name', 'column2')
                ->addFloatField('sql_table_name', 'column3')
                ->addBlindIndex(
                    'sql_table_name',
                    'column1',
                    new BlindIndex('sql_table_name_column1_index_1', [], 8)
                )
                ->addCompoundIndex(
                    'sql_table_name',
                     (new CompoundIndex(
                         'sql_table_name_compound',
                         ['column1', 'column2'],
                         4,
                         true
                     ))->addTransform('column1', new LastFourDigits())
                )
            ->addTable('other_table');
    }
}
```

If you're not familiar with the `EncryptedMultiRows` API, please refer to the
relevant section of the [CipherSweet documentation](https://github.com/paragonie/ciphersweet/tree/master/docs#encryptedmultirows).

## Storing and Searching on Encrypted Data