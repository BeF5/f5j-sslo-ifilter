i-FILTERにてHTTP/HTTPS判別するためのiRuleの作成
==================================================================

　i-FILTER PROXY版は、HTTPのリクエストヘッダでHTTPサーバへの通信かHTTPSサーバへの通信かを判別しています。i-FILTERがHTTP/HTTPS判別可能となるためのiRuleを作成します。**X-Forwarded-Proto** というヘッダーに利用プロトコル（http or https）を加えて、HTTPリクエストに挿入します。ここではあわせて、クライアントのIPアドレスも **X-Forwarded-For** に挿入しています。

#. **Local Traffic >> iRules** にて、:guilabel:`Create` ボタンを押します。**任意の名前** を入力して、**Definition** に以下サンプルiRuleを入力し、:guilabel:`Finished` ボタンを押します。（以下のiRuleはあくまでもサンプルとなります。同じ主旨の内容であれば下記と同じでなくても構いません。）

    例）HTTPリクエストにi-FILTER連携のヘッダを挿入するためのiRuleサンプル  

    .. code-block:: bash

            ###  Add this iRule to Virtual Server:ssloS_XXXX-t-4 ###

            when HTTP_REQUEST {

                if { [TCP::local_port] == 80 } {
                    # For HTTP protocol
                    HTTP::header replace "X-Forwarded-Proto" "http"     

                } elseif { [TCP::local_port] == 443 } {
                    # For HTTPS protocl
                    HTTP::header replace "X-Forwarded-Proto" "https"
                } 
    
                sharedvar ctx
                # For Client IP address
                lappend ctx(headers) "X-Forwarded-For" [IP::client_addr]

                if { [expr { [llength $ctx(headers)] % 2 }] == 0 } { 
                    foreach {h_name h_value} $ctx(headers) {
                        HTTP::header replace ${h_name} ${h_value} 
                    }
                }
            }    
    | 接続テストをされる際には、変数情報をログ出力するなどして意図する値が入っているか確かめて頂くことをおすすめします。
    
    | 例１）ヘッダー内容を確認するデバッグ
    | foreach attr "[HTTP::header names]" {
    |     log local0. "$attr : [HTTP::header value $attr]"
    | }
