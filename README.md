# Обоснование
## О проекте
В данном небольшом документе я бы хотел описать идею проекта, выбор подходов, конкурентов и иные показатели, которые привели меня к написанию его таким, какой он есть.

## Экономическое обоснование
Исследовав несколько ресурсов сети интернет, я нашёл два проекта, которые “конкурируют” с моим:
- https://habr.com/ru/company/mixbytes/blog/420941/
- https://www.taidekivi.com/node/31


Как можно заметить, в обоих случаях идея сводится к тому, что итог спора определяется человеком – в первом случае это арбитр, во втором – сам организатор. Размышляя над идеей проверки результатов, я пришёл к выводу, что имею две возможных опции: человек-арбитр или оракул. По итогу размышлений от оракулов было решено отказаться, так как сферы споров будут ограничены функционалом оракулов, или же – техническими умениями пользователя по предоставлению API, вследствие чего выбор был сделан в пользу арбитра. Тем не менее, один арбитр действительно может быть легко подкуплен, что сводит на нет смысл спора. Во избежание этой проблемы количество арбитров в моём проекте было увеличено до трёх, их фиксированной ставкой стало получение 7% от общего банка ставок, а сам процесс голосования был реализован через паттерн commit-reveal. Сервис также берёт комиссию в 7%, а игроки смогут сделать ставку только после завершения процесса подключения судей. Игроки могут увидеть список самих игроков, хоста и судей. Деньги распределяются сразу после reveal-фазы без человеческого участия. Почему это должно сработать? Во-первых, “жулику” придётся подкупить как минимум двух судей, что уже оказывается финансово невыгодным: разумеется, ты теряешь всё в случае проигрыша, но выигрыш в случае игры как минимум вдесятером сводится буквально к проценту-двум. Более того, судьи при такой довольно крупной комиссии будут стремиться играть по правилам, чтобы сохранить доступ к заработку. Игроки при регистрации смогут увидеть адреса уже известных мошенников и просто откажутся от участия в игре с бесчестными судьями. Более того, до момента reveal-фазы “жулик” не сможет узнать, какой голос отдал судья, но уже после не сможет ни на что повлиять, если судьи его обманули, забрав взятку, но сыграв честно: деньги распределятся автоматически, на что сам мошенник не сможет повлиять. 
Таким образом, идея выделяет ряд мотиваций и систем сдерживания для того, чтобы свести жульничество к минимуму. Возникает резонное замечание: “Но ведь они всё ещё могут рискнуть и сжульничать!” К сожалению, это правда, но в нашей жизни довольно мало систем с надёжностью в 100%. Тем не менее, данная концепция позволяет минимизировать риски подлога. Даже если он и случится, то “жулик” или “жулики” в дальнейшем окажутся изгоями: как игроки, так и судьи. 
Как и было сказано выше, ни один судья не знает, как проголосовал другой, до reveal-этапа. Разумеется, к этому моменту все голоса уже записаны на commit-фазе, и ничего подменить не получится. Судьи не получат свою комиссию до тех пор, пока последний судья не проголосует, поэтому каждый из них будет стремиться как можно быстрее раскрыть свой голос. Комиссия фиксирована и не зависит от решения арбитров.
Таким образом, смарт-контракт предлагает достаточно устойчивую финансовую схему.
## Описание работы контракта

Сначала создаётся контракт и назначается хост: хост передаёт средства на счёт контракту, устанавливает условие, количество игроков и дату со старта, к которой оно должно быть проверено. По условию контракта, хост размещает ставку в пользу своего условия. После этого назначаются судьи. Далее игроки начинают присоединяться к игре: они могут поставить ровно столько же, сколько и хост. Если они пришлют больше, разница им вернётся(нам чужого не надо). Судья, разумеется, не может ставить. Как только все собраны, хост начинает игру. С этого момента идёт отсчёт до даты, когда судьи смогут проголосовать. После истечения срока ожидания судьи вносят зашифрованные голоса. Как только последний судья внесёт голос, открывается фаза раскрытия голоса. После раскрытия выигрывает вариант, за который проголосовало минимум два судьи. Сразу после начинается перевод средств путём деления суммы после вычета комиссии на число победивших игроков.