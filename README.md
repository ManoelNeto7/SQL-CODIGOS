# SQL-CODIGOS
CODIGOS USUAIS SQL

PROCEDURE ALTER VIEW

ALTER VIEW vw_atp_mercado AS

WITH base AS (

    SELECT

        UPPER(
            CONCAT(
                YEAR(A.DataEmissao_datetime),
                MONTH(A.DataEmissao_datetime),
                A.Trecho,

                CASE A.ApelidoFornecedor
                    WHEN 'GOL LINHAS AÉREAS S.A.' THEN 'GOL'
                    WHEN 'GOL LINHAS AÉREAS' THEN 'GOL'
                    WHEN 'GOL LINHAS AEREAS S.A.' THEN 'GOL'
                    WHEN 'GOL LINHAS AEREAS' THEN 'GOL'
                    WHEN 'PASSAREDO LINHAS AEREAS' THEN 'PASSAREDO'
                    WHEN 'Latam Airlines Brasil' THEN 'LATAM'
                    WHEN 'Gol Transportes Aereos S.A.' THEN 'GOL'
                    ELSE A.ApelidoFornecedor
                END,

                CASE
                    WHEN B.aereo_tarifa_classe IS NULL THEN 'ECONÔMICA'
                    WHEN B.aereo_tarifa_classe = 'ECONOMICA' THEN 'ECONÔMICA'
                    ELSE B.aereo_tarifa_classe
                END,

                A.TipoRota,

                CASE
                    WHEN DATEDIFF(
                        DAY,
                        A.DataEmissao_datetime,
                        A.DataIn_datetime
                    ) < 0 THEN 0
                    ELSE DATEDIFF(
                        DAY,
                        A.DataEmissao_datetime,
                        A.DataIn_datetime
                    )
                END
            )
        ) AS Personalizar,

        A.Tarifa AS [ATP MERCADO]

    FROM fato_vendas A

    LEFT JOIN aereo_wide_v2 B
        ON CAST(A.Requisicao AS VARCHAR) =
           CAST(B.solicitacao_id AS VARCHAR)

    WHERE A.Reemissao = 'N'
      AND A.DescricaoProduto = 'Aéreo'
      AND NomeTipoMiscelanio = 'None'
      AND A.Situacao = 1
      AND YEAR(A.DataEmissao_datetime) >= YEAR(GETDATE()) - 1
      AND A.Tarifa > 400
      AND TipoCliente = 'Juridica'
      AND PCC NOT LIKE '%CENTRAL DE EVENTOS%'
      AND NomeTipoMiscelanio NOT LIKE '%FEE%'
)

SELECT
    Personalizar,
    AVG([ATP MERCADO]) AS [ATP MERCADO]

FROM base

WHERE [ATP MERCADO] > 400

GROUP BY Personalizar;
