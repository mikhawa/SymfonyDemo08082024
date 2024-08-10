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
// puis
    #[ORM\Column(type: Types::TEXT)]
    #[Assert\NotBlank(message: 'comment.blank')]
    #[Assert\Length(min: 5, minMessage: 'comment.too_short', max: 10000, maxMessage: 'comment.too_long')]
    private ?string $content = null;
// Par
    #[ORM\Column(type: Types::TEXT, nullable: true)]
    #[Assert\NotBlank(message: 'comment.blank')]
    #[Assert\Length(min: 5, minMessage: 'comment.too_short', max: 1000, maxMessage: 'comment.too_long')]
    private ?string $content = null;
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
```

### Mise à jour de la base de donnée

Nous allons mettre à jour la base de donnée en utilisant `MakerBundle`

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

Syntax error or access violation : 1071 La clé est trop longue. Longueur maximale : 1000

### copilot solution

L'erreur 1071 de MySQL 8 se produit généralement lorsque la longueur des index dépasse la limite maximale autorisée (767 bytes pour les index InnoDB). Cette erreur est courante lorsque l'on crée des index sur des colonnes de type `VARCHAR` avec un jeu de caractères UTF-8, car chaque caractère peut nécessiter jusqu'à 4 bytes.

Voici quelques solutions pour résoudre ce problème dans votre migration :

1. **Réduire la longueur des colonnes indexées** :
   - Spécifier une longueur maximale pour les colonnes `VARCHAR` indexées.

2. **Utiliser un jeu de caractères plus compact** :
   - Utiliser `utf8mb4` au lieu de `utf8mb4_unicode_ci`, ou même `utf8` si vous n'avez pas besoin de l'intégralité du jeu de caractères UTF-8.

3. **Changer le type de colonne** :
   - Pour certaines colonnes, vous pouvez utiliser un type de données différent si approprié.

Voici une version modifiée de votre migration qui applique la première solution :

```php
public function up(Schema $schema): void
{
    // this up() migration is auto-generated, please modify it to your needs
    $this->addSql('CREATE TABLE symfony_demo_comment (
        id INT UNSIGNED AUTO_INCREMENT NOT NULL,
        content LONGTEXT NOT NULL,
        published_at DATETIME NOT NULL,
        post_id INT UNSIGNED NOT NULL,
        author_id INT UNSIGNED NOT NULL,
        PRIMARY KEY(id)
    ) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');

    $this->addSql('CREATE TABLE symfony_demo_post (
        id INT UNSIGNED AUTO_INCREMENT NOT NULL,
        title VARCHAR(255) NOT NULL,
        slug VARCHAR(191) NOT NULL, -- Limiter la longueur à 191 caractères
        summary VARCHAR(255) NOT NULL,
        content LONGTEXT NOT NULL,
        published_at DATETIME NOT NULL,
        author_id INT UNSIGNED NOT NULL,
        PRIMARY KEY(id),
        UNIQUE INDEX UNIQ_5A8A6C8D989D9B62 (slug) -- Index sur slug
    ) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');

    $this->addSql('CREATE TABLE symfony_demo_post_tag (
        post_id INT UNSIGNED NOT NULL,
        tag_id INT UNSIGNED NOT NULL,
        INDEX IDX_6ABC1CC44B89032C (post_id),
        INDEX IDX_6ABC1CC4BAD26311 (tag_id),
        PRIMARY KEY(post_id, tag_id)
    ) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');

    $this->addSql('CREATE TABLE symfony_demo_tag (
        id INT UNSIGNED AUTO_INCREMENT NOT NULL,
        name VARCHAR(191) NOT NULL, -- Limiter la longueur à 191 caractères
        UNIQUE INDEX UNIQ_4D5855405E237E06 (name),
        PRIMARY KEY(id)
    ) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');

    $this->addSql('CREATE TABLE symfony_demo_user (
        id INT UNSIGNED AUTO_INCREMENT NOT NULL,
        full_name VARCHAR(255) NOT NULL,
        username VARCHAR(191) NOT NULL, -- Limiter la longueur à 191 caractères
        email VARCHAR(191) NOT NULL, -- Limiter la longueur à 191 caractères
        password VARCHAR(255) NOT NULL,
        PRIMARY KEY(id),
        UNIQUE INDEX UNIQ_4D585540F85E0677 (username),
        UNIQUE INDEX UNIQ_4D585540E7927C74 (email)
    ) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');

    $this->addSql('ALTER TABLE symfony_demo_comment ADD CONSTRAINT FK_53AD8F834B89032C FOREIGN KEY (post_id) REFERENCES symfony_demo_post (id)');
    $this->addSql('ALTER TABLE symfony_demo_comment ADD CONSTRAINT FK_53AD8F83F675F31B FOREIGN KEY (author_id) REFERENCES symfony_demo_user (id)');
    $this->addSql('ALTER TABLE symfony_demo_post ADD CONSTRAINT FK_58A92E65F675F31B FOREIGN KEY (author_id) REFERENCES symfony_demo_user (id)');
    $this->addSql('ALTER TABLE symfony_demo_post_tag ADD CONSTRAINT FK_6ABC1CC44B89032C FOREIGN KEY (post_id) REFERENCES symfony_demo_post (id) ON DELETE CASCADE');
    $this->addSql('ALTER TABLE symfony_demo_post_tag ADD CONSTRAINT FK_6ABC1CC4BAD26311 FOREIGN KEY (tag_id) REFERENCES symfony_demo_tag (id) ON DELETE CASCADE');
}
```

En limitant la longueur des colonnes `VARCHAR` à 191 caractères (ou moins si possible), vous pouvez éviter de dépasser la limite maximale de longueur des index en MySQL 8.

[1]: https://symfony.com/doc/current/best_practices.html
[2]: https://symfony.com/doc/current/setup.html#technical-requirements
[3]: https://symfony.com/doc/current/setup/web_server_configuration.html
[4]: https://symfony.com/download
[5]: https://symfony.com/book
[6]: https://getcomposer.org/