.. index::
   single: Configuration; Doctrine DBAL
   single: Doctrine; DBAL configuration

Configura��o
=============

Um exemplo de configura��o com banco de dados MySQL pode parecer como o 
seguinte exemplo:

.. code-block:: yaml

    # app/config/config.yml
    doctrine:
        dbal:
            connections:
                default:
                    driver:   pdo_mysql
                    dbname:   Symfony2
                    user:     root
                    password: null

O DoctrineBundle suporta todos os par�metros que todos os drivers padr�es do doctrine
aceitam, convertidos para os padr�es de nomenclatura do XML ou YAML que o Symfony aplica.
Veja a `documentation`_ do Doctrine DBAL para mais informa��es. Al�m disso, existem algumas 
op��es relacionadas ao Symfony que voc� pode configurar. O bloco a seguir mostra todas
as chaves de configura��o poss�veis, sem maiores explicaca��es de seus significados.

.. configuration-block::

    .. code-block:: yaml

        doctrine:
            dbal:
                connections:
                    default:
                        dbname:               database
                        host:                 localhost
                        port:                 1234
                        user:                 user
                        password:             secret
                        driver:               pdo_mysql
                        driver_class:         MyNamespace\MyDriverImpl
                        options:
                            foo: bar
                        path:                 %kernel.data_dir%/data.sqlite
                        memory:               true
                        unix_socket:          /tmp/mysql.sock
                        wrapper_class:        MyDoctrineDbalConnectionWrapper
                        charset:              UTF8
                        logging:              %kernel.debug%
                        platform_service:     MyOwnDatabasePlatformService

    .. code-block:: xml

        <!-- xmlns:doctrine="http://symfony.com/schema/dic/doctrine" -->
        <!-- xsi:schemaLocation="http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd"> -->

        <doctrine:config>
            <doctrine:dbal>
                <doctrine:connection
                    name="default"
                    dbname="database"
                    host="localhost"
                    port="1234"
                    user="user"
                    password="secret"
                    driver="pdo_mysql"
                    driver-class="MyNamespace\MyDriverImpl"
                    path="%kernel.data_dir%/data.sqlite"
                    memory="true"
                    unix-socket="/tmp/mysql.sock"
                    wrapper-class="MyDoctrineDbalConnectionWrapper"
                    charset="UTF8"
                    logging="%kernel.debug%"
                    platform-service="MyOwnDatabasePlatformService"
                />
            </doctrine:dbal>
        </doctrine:config>

Tamb�m existem um monte de par�metros do dependency injection container
que permitem a voc� especificar quais classes s�o usadas (com seus valores padr�es):

.. code-block:: yaml

    parameters:
        doctrine.dbal.logger_class: Symfony\Bundle\DoctrineBundle\Logger\DbalLogger
        doctrine.dbal.configuration_class: Doctrine\DBAL\Configuration
        doctrine.data_collector.class: Symfony\Bundle\DoctrineBundle\DataCollector\DoctrineDataCollector
        doctrine.dbal.event_manager_class: Doctrine\Common\EventManager
        doctrine.dbal.events.mysql_session_init.class: Doctrine\DBAL\Event\Listeners\MysqlSessionInit
        doctrine.dbal.events.oracle_session_init.class: Doctrine\DBAL\Event\Listeners\OracleSessionInit

Se voc� quer configurar multiplas conex�es, voc� pode fazer isso simplesmente listando
elas sob a chave ``connections``. Todos os par�metros mostrados acima podem
tamb�m ser especificados nas sub-chaves da conex�o.

.. code-block:: yaml

    doctrine:
        dbal:
            default_connection:       default
            connections:
                default:
                    dbname:           Symfony2
                    user:             root
                    password:         null
                    host:             localhost
                customer:
                    dbname:           customer
                    user:             root
                    password:         null
                    host:             localhost

Se voc� tem definidas multiplas conex�es, voc� pode usar o 
``$this->get('doctrine.dbal.[connectionname]_connection')`` tamb�m, 
mas voc� deve passar a ele um argumento com o nome 
da conex�o que voc� quer ter

.. code-block:: php

    class UserController extends Controller
    {
        public function indexAction()
        {
            $defaultConn1 = $this->get('doctrine.dbal.connection');
            $defaultConn2 = $this->get('doctrine.dbal.default_connection');
            // $defaultConn1 === $defaultConn2

            $customerConn = $this->get('doctrine.dbal.customer_connection');
        }
    }

.. _documentation: http://www.doctrine-project.org/docs/dbal/2.0/en
