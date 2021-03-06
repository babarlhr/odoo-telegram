.. image:: https://itpp.dev/images/infinity-readme.png
   :alt: Tested and maintained by IT Projects Labs
   :target: https://itpp.dev

==============
 Telegram bot
==============

The module apply monkey patch for ``PreforkServer`` in order to run new process ``WorkerTelegram``, which run threads as following:

* ``WorkerTelegram`` (1 process)

  * ``odoo_dispatch`` (1 thread)

    * Instance of ``TelegramDispatch`` (evented tread to listen and notify about events from other odoo processes).
    * To send events the function ``registry['telegram.bus'].sendone()`` is used

  * ``OdooTelegramThread`` (1 thread per each database)

    * Listen for events from ``odoo_dispatch`` and delegate work to ``odoo_thread_pool``
    * ``odoo_thread_pool`` (1 pool)

      * Pool of threads to handle odoo events.
      * Handles updates of telegram parameters 
      * Executes ``registry['telegram.command'].odoo_listener()``
      * Has threads number equal to ``telegram.num_odoo_threads`` parameter.

  * ``BotPollingThread``  (1 thread per each database with token)

    * Calls ``bot.polling()`` function
    * telebot's ``polling_thread`` (1 thread)

      * Listen for events from Telegram and delegate work to telebot's ``worker_pool``

    * telebot's ``worker_pool`` (1 pool)

      * Pool of thread to handle telegram events.
      * Executes ``registry['telegram.command'].telegram_listener()``.
      * Has threads number equal to ``telegram.num_telegram_threads`` parameter.

Further information
-------------------

Odoo Apps Store: https://apps.odoo.com/apps/modules/9.0/telegram


Tested on `Odoo 9.0 <https://github.com/odoo/odoo/commit/d3dd4161ad0598ebaa659fbd083457c77aa9448d>`_
