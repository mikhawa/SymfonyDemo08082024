Symfony Demo Application
========================

The "Symfony Demo Application" is a reference application created to show how
to develop applications following the [Symfony Best Practices][1].

You can also learn about these practices in [the official Symfony Book][5].

Requirements
------------

  * PHP 8.2.0 or higher;
  * PDO-SQLite PHP extension enabled;
  * and the [usual Symfony application requirements][2].

Installation
------------

There are 3 different ways of installing this project depending on your needs:

**Option 1.** [Download Symfony CLI][4] and use the `symfony` binary installed
on your computer to run this command:

```bash
symfony new --demo my_project
```

**Option 2.** [Download Composer][6] and use the `composer` binary installed
on your computer to run these commands:

```bash
# you can create a new project based on the Symfony Demo project...
composer create-project symfony/symfony-demo my_project

# ...or you can clone the code repository and install its dependencies
git clone https://github.com/symfony/demo.git my_project
cd my_project/
composer install
```

**Option 3.** Click the following button to deploy this project on Platform.sh,
the official Symfony PaaS, so you can try it without installing anything locally:

<p align="center">
<a href="https://console.platform.sh/projects/create-project?template=https://raw.githubusercontent.com/symfonycorp/platformsh-symfony-template-metadata/main/symfony-demo.template.yaml&utm_content=symfonycorp&utm_source=github&utm_medium=button&utm_campaign=deploy_on_platform"><img src="https://platform.sh/images/deploy/lg-blue.svg" alt="Deploy on Platform.sh" width="180px" /></a>
</p>

Usage
-----

There's no need to configure anything before running the application. There are
2 different ways of running this application depending on your needs:

**Option 1.** [Download Symfony CLI][4] and run this command:

```bash
cd my_project/
symfony serve
```

Then access the application in your browser at the given URL (<https://localhost:8000> by default).

**Option 2.** Use a web server like Nginx or Apache to run the application
(read the documentation about [configuring a web server for Symfony][3]).

On your local machine, you can run this command to use the built-in PHP web server:

```bash
cd my_project/
php -S localhost:8000 -t public/
```

Tests
-----

Execute this command to run tests:

```bash
cd my_project/
./bin/phpunit
```



## Migration vers MySQL 8

### Création du fichier `.env.local`

Nous allons dupliquer le fichier `.env` sous le nom `.env.local`
(il sera ignoré sur `Github`)

Puis modifier le chemin vers la base de donnée MySQL :

```dotenv
#  * .env.local          uncommitted file with local overrides
# ...
# Base de donnée originale en sqlite
# DATABASE_URL=sqlite:///%kernel.project_dir%/data/database.sqlite
# Nouvelle base de donnée en MySQL 8.2
DATABASE_URL="mysql://root:@127.0.0.1:3306/demo_symfony_7?serverVersion=8.2&charset=utf8mb4"
# ...
```

### Création de la base de donnée

Nous allons créer la base de donnée en utilisant `Doctrine`

```bash
php bin/console doctrine:database:create 
```

### Modification des entités

Pour optimiser le passage de SQLite vers MySQL, nous allons modifier légèrement les entités.

Nous allons mettre les clés primaires en `unsigned` et limiter la longueur des colonnes `VARCHAR` qui servent de clefs étrangères à 191 caractères pour éviter l'erreur `MySQL 1071` .

Ceci permet d'éviter de dépasser la limite maximale de longueur des index en MySQL 8.

#### `src/Entity/Comment.php`

```php
// On remplace
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: Types::INTEGER)]
    private ?int $id = null;
// Par
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: Types::INTEGER, 
    options: ["unsigned" => true])]
    private ?int $id = null;

```

#### `src/Entity/Post.php`

```php
// On remplace
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: Types::INTEGER)]
    private ?int $id = null;
// Par
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: Types::INTEGER, 
    options: ["unsigned" => true])]
    private ?int $id = null;
// puis
    #[ORM\Column(type: Types::STRING)]
    private ?string $slug = null;
// Par
    #[ORM\Column(type: Types::STRING, 
    options: ["length" => 191])]
    private ?string $slug = null;

```

#### `src/Entity/User.php`

```php
// On remplace
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: Types::INTEGER)]
    private ?int $id = null;
// Par
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: Types::INTEGER, 
    options: ["unsigned" => true])]
    private ?int $id = null;
// puis
    #[ORM\Column(type: Types::STRING, unique: true)]
    #[Assert\NotBlank]
    #[Assert\Length(min: 2, max: 50)]
    private ?string $username = null;
// Par
    #[ORM\Column(type: Types::STRING, 
    unique: true,
    options: ["length" => 191])]
    #[Assert\NotBlank]
    #[Assert\Length(min: 2, max: 50)]
    private ?string $username = null;
// puis
    #[ORM\Column(type: Types::STRING, unique: true)]
    #[Assert\NotBlank]
    #[Assert\Email]
    private ?string $email = null;
// Par
    #[ORM\Column(type: Types::STRING, 
    unique: true,
    options: ["length" => 191])]
    #[Assert\NotBlank]
    #[Assert\Email]
    private ?string $email = null;

```

#### `src/Entity/Tag.php`

```php
// On remplace
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: Types::INTEGER)]
    private ?int $id = null;
// Par
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: Types::INTEGER, 
    options: ["unsigned" => true])]
    private ?int $id = null;
// puis
    #[ORM\Column(type: Types::STRING,unique: true)]
    private readonly string $name;
// Par
    #[ORM\Column(type: Types::STRING,
    unique: true,
    options: ["length" => 191])]
    private readonly string $name;
```

### Mise à jour de la base de donnée

Nous allons mettre à jour la base de donnée en utilisant `MakerBundle`

```bash
php bin/console make:migration
```

Nous allons ensuite ouvrir le fichier de migration généré et ajouter les options d'encodage `utf8mb4` et de `collation` `utf8mb4_unicode_ci` pour chaque table, ainsi que le moteur `InnoDB`.

```DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB' pour chaque table.```

```php
# migrations/Version20240810071705.php
// ...
    public function up(Schema $schema) : void
    {
        // this up() migration is auto-generated, please modify it to your needs
        $this->addSql('CREATE TABLE symfony_demo_comment (id INT UNSIGNED AUTO_INCREMENT NOT NULL, content LONGTEXT NOT NULL, published_at DATETIME NOT NULL, post_id INT UNSIGNED NOT NULL, author_id INT UNSIGNED NOT NULL, INDEX IDX_53AD8F834B89032C (post_id), INDEX IDX_53AD8F83F675F31B (author_id), PRIMARY KEY(id)) 
        DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');

        $this->addSql('CREATE TABLE symfony_demo_post (id INT UNSIGNED AUTO_INCREMENT NOT NULL, title VARCHAR(255) NOT NULL, slug VARCHAR(191) NOT NULL, summary VARCHAR(255) NOT NULL, content LONGTEXT NOT NULL, published_at DATETIME NOT NULL, author_id INT UNSIGNED NOT NULL, INDEX IDX_58A92E65F675F31B (author_id), PRIMARY KEY(id)) 
        DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');

        $this->addSql('CREATE TABLE symfony_demo_post_tag (post_id INT UNSIGNED NOT NULL, tag_id INT UNSIGNED NOT NULL, INDEX IDX_6ABC1CC44B89032C (post_id), INDEX IDX_6ABC1CC4BAD26311 (tag_id), PRIMARY KEY(post_id, tag_id)) 
        DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');

        $this->addSql('CREATE TABLE symfony_demo_tag (id INT UNSIGNED AUTO_INCREMENT NOT NULL, name VARCHAR(191) NOT NULL, UNIQUE INDEX UNIQ_4D5855405E237E06 (name), PRIMARY KEY(id)) 
        DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');

        $this->addSql('CREATE TABLE symfony_demo_user (id INT UNSIGNED AUTO_INCREMENT NOT NULL, full_name VARCHAR(255) NOT NULL, username VARCHAR(191) NOT NULL, email VARCHAR(191) NOT NULL, password VARCHAR(255) NOT NULL, roles JSON NOT NULL, UNIQUE INDEX UNIQ_8FB094A1F85E0677 (username), UNIQUE INDEX UNIQ_8FB094A1E7927C74 (email), PRIMARY KEY(id)) 
        DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');
# ...
```

Nous allons ensuite exécuter la migration

```bash
php bin/console doctrine:migrations:migrate
```

### Ajout de données

Nous allons ajouter des données dans la base de donnée

```bash
php bin/console doctrine:fixtures:load
```

### Vérification

Nous allons vérifier que tout fonctionne correctement

```bash
symfony serve -d
```

[1]: https://symfony.com/doc/current/best_practices.html
[2]: https://symfony.com/doc/current/setup.html#technical-requirements
[3]: https://symfony.com/doc/current/setup/web_server_configuration.html
[4]: https://symfony.com/download
[5]: https://symfony.com/book
[6]: https://getcomposer.org/