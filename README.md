# git-ssh-key-action
GitHub Action that select right ssh key per repository.

This is useful when you have multiple private repositories and you want to use different ssh deploy keys for each of them.

## Usage

Each SSH key must be present under `~/.ssh` folder and should have the name of the corresponding repository. The `/` must be converted to a `-`.

For example the key file for the repository `TestRepository/my-first-repo` should be `~/.ssh/TestRepository-my-first-repo`.  

You then just need to add the following step to your workflow:

```yaml
  - name: Select SSH key per repository
    uses: Flowster/git-ssh-key-action@v1
```

## Example

You can use it in combination with an action as [shimtaro/ssh-key-action](https://github.com/shimataro/ssh-key-action).

```yaml
name: Code Quality
on: [ pull_request ]
jobs:
  symfony:
    name: Symfony 6.2 (PHP ${{ matrix.php-versions }})
    runs-on: ubuntu-latest
    strategy:
      fail-fast: true
      matrix:
        php-versions: [ '8.1' ]
    steps:
      # —— Setup Github actions 🐙 —————————————————————————————————————————————
      - name: Checkout
        uses: actions/checkout@v2

      - name: Install SSH key - Private Repository Flowster/test1
        uses: shimataro/ssh-key-action@v2
        with:
          key: ${{ secrets.SSH_KEY_TEST1 }}
          name: Flowster-test1
          known_hosts: ${{ secrets.KNOWN_HOSTS }}

      - name: Install SSH key - Private Repository Flowster/test2
        uses: shimataro/ssh-key-action@v2
        with:
          key: ${{ secrets.SSH_KEY_TEST2 }}
          name: Flowster-test2
          known_hosts: ${{ secrets.KNOWN_HOSTS }}

      - name: Select SSH key per repository
        uses: Flowster/git-ssh-key-action@v1

      - name: Setup PHP, extensions and composer with shivammathur/setup-php
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-versions }}
          extensions: mbstring, xml, ctype, iconv, intl, pdo, pdo_mysql, dom, filter, gd, iconv, json, mbstring
        env:
          update: true

      # —— Composer 🧙‍️ —————————————————————————————————————————————————————————
      - name: Install Composer dependencies # From private repositories
        run: composer install

      # —— Static analysis ✨ —————————————————————————————————————————————————
      - name: PHP-CS-Fixer
        run: ./vendor/bin/php-cs-fixer fix --dry-run --diff --using-cache=no

      - name: PHPStan
        run: ./vendor/bin/phpstan analyse .
```

## Credits

This action is based on [custom_keys_git_ssh](https://gist.github.com/vhermecz/4e2ae9468f2ff7532bf3f8155ac95c74) gist by [vhermecz](https://gist.github.com/vhermecz).