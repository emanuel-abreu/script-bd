# script-bd

## 1. Veículos cuja marca inicia com 'F' e custam até R$ 50 000,00
SELECT *
FROM Veiculo
WHERE marca LIKE 'F%' 
  AND preco <= 50000.00;


## 2. Quantidade de clientes com idade ≤ 35 anos
SELECT COUNT(*) AS qtde_clientes_ate_35
FROM Cliente
WHERE idade <= 35;


## 3. Nomes e CPFs dos clientes que não realizaram nenhuma compra
SELECT c.nome, c.cpf
FROM Cliente c
LEFT JOIN Venda v ON c.id = v.id_cliente
WHERE v.id IS NULL;


## 4. Média das compras por cidade
--    (para cada cidade, primeiro soma o total comprado por cliente e depois faz a média desses totais)
WITH total_por_cliente AS (
  SELECT id_cliente, SUM(ve.preco) AS total_compras
  FROM Venda v
  JOIN Veiculo ve ON v.id_veiculo = ve.id
  GROUP BY id_cliente
)
SELECT c.cidade,
       AVG(tc.total_compras) AS media_compras_por_cliente
FROM Cliente c
LEFT JOIN total_por_cliente tc ON c.id = tc.id_cliente
GROUP BY c.cidade;


## 5. Clientes que somente compraram de um único vendedor
SELECT c.nome
FROM Cliente c
JOIN Venda v ON c.id = v.id_cliente
GROUP BY c.id, c.nome
HAVING COUNT(DISTINCT v.id_vendedor) = 1;


## 6. Nome, marca, ano do veículo e quantidade vendida (exclui não vendidos)
SELECT ve.modelo,
       ve.marca,
       ve.ano,
       COUNT(*) AS unidades_vendidas
FROM Veiculo ve
JOIN Venda v ON ve.id = v.id_veiculo
GROUP BY ve.id, ve.modelo, ve.marca, ve.ano;


## 7. Nomes dos clientes e total gasto em compras (clientes sem compras aparecem com 0), ordenado decrescente
SELECT c.nome,
       COALESCE(SUM(ve.preco), 0) AS total_compras
FROM Cliente c
LEFT JOIN Venda v ON c.id = v.id_cliente
LEFT JOIN Veiculo ve ON v.id_veiculo = ve.id
GROUP BY c.nome
ORDER BY total_compras DESC;


## 8. Número de compras por cliente da cidade de Russas, ordenado decrescente
SELECT c.nome,
       COUNT(v.id) AS num_vendas
FROM Cliente c
LEFT JOIN Venda v ON c.id = v.id_cliente
WHERE c.cidade = 'Russas'
GROUP BY c.nome
ORDER BY num_vendas DESC;


## 9. Veículos (nome, marca, ano) vendidos por mais de um vendedor
SELECT ve.modelo,
       ve.marca,
       ve.ano
FROM Veiculo ve
JOIN Venda v ON ve.id = v.id_veiculo
GROUP BY ve.id, ve.modelo, ve.marca, ve.ano
HAVING COUNT(DISTINCT v.id_vendedor) > 1;


## 10. Nome do vendedor e número de vendas em junho/2024 (quem não vendeu aparece com 0)
WITH vendas_junho AS (
  SELECT id_vendedor,
         COUNT(*) AS qtde_vendas
  FROM Venda
  WHERE data BETWEEN '2024-06-01' AND '2024-06-30'
  GROUP BY id_vendedor
)
SELECT vd.nome,
       COALESCE(vj.qtde_vendas, 0) AS vendas_em_junho
FROM Vendedor vd
LEFT JOIN vendas_junho vj ON vd.id = vj.id_vendedor;
