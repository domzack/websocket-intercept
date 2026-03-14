// ==UserScript==
// @name         Agressive
// @namespace    http://tampermonkey.net/
// @version      1.2
// @description  Intercepta TODAS as conexões WebSocket e filtra mensagens com volume
// @author       Rodrigo
// @match        *://*/*
// @grant        none
// @run-at       document-start
// ==/UserScript==

(function() {
    'use strict';

    let ultimoTimestamp = null;
    let ultimoVolume = 0;
    let ultimoClose = null;

    function hookWebSocket() {
        if (window.WebSocket.__IS_HOOKED__) return;

        const NativeWebSocket = window.WebSocket;
        window.WebSocket = function(url, protocols) {
            const ws = new NativeWebSocket(url, protocols);

            ws.addEventListener('message', (event) => {
                const raw = event.data;
                const partes = raw.split("~m~");

                for (let trecho of partes) {
                    if (trecho.trim().startsWith("{")) {
                        try {
                            const obj = JSON.parse(trecho);

                            // --- Candle update (du) ---
                            if (obj.m === "du") {
                                const payload = obj.p[1];
                                for (const key in payload) {
                                    const serie = payload[key];
                                    if (serie.s && serie.s.length > 0) {
                                        const v = serie.s[0].v;
                                        const timestamp = v[0];
                                        const close = v[4];
                                        const volume = v[5];

                                        // Novo candle → reset volume
                                        if (ultimoTimestamp !== timestamp) {
                                            ultimoTimestamp = timestamp;
                                            ultimoVolume = 0;
                                        }

                                        // Delta volume
                                        const deltaVolume = volume - ultimoVolume;

                                        // Determina tipo de agressão
                                        let tipoAgressao = "----";
                                        if (ultimoClose !== null) {
                                            if (close > ultimoClose) {
                                                tipoAgressao = "BUY ";
                                            } else if (close < ultimoClose) {
                                                tipoAgressao = "SELL";
                                            } else {
                                                // Se não mudou, tenta avaliar pelo bid/ask
                                                const valores = obj?.p?.[1]?.v;
                                                if (valores) {
                                                    const bid = valores.bid;
                                                    const ask = valores.ask;
                                                    if (close >= ask) {
                                                        tipoAgressao = "BUY ";
                                                    } else if (close <= bid) {
                                                        tipoAgressao = "SELL";
                                                    }
                                                }
                                            }
                                        }

                                        // Imprime se houve mudança
                                        if (deltaVolume > 0 || close !== ultimoClose) {
                                            const _now = Math.floor(Date.now()/1);
                                            console.log({
                                                timestamp: _now,
                                                close,
                                                agressao: tipoAgressao,
                                                volume: deltaVolume
                                            });
                                            ultimoVolume = volume;
                                            ultimoClose = close;
                                        }
                                    }
                                }
                            }

                            // --- Quote update (qsd) ---
                            if (obj.m === "qsd") {
                                // opcional: atualizar bid/ask para uso posterior
                                const valores = obj?.p?.[1]?.v;
                                if (valores) {
                                    // mantemos bid/ask disponíveis
                                }
                            }

                        } catch (e) {
                            // ignora erros
                        }
                    }
                }
            });

            return ws;
        };

        window.WebSocket.__IS_HOOKED__ = true;
        console.log("Interceptor ativo: imprime {timestamp, close, deltavolume, tipo_agressao}");
    }

    hookWebSocket();
    [1000, 3000, 8000].forEach(ms => setTimeout(hookWebSocket, ms));
})();
