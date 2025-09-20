
2.  ## CRICITICAL- 1 Protocol wrongly burns from `addressthis` instead of `to` tokens would an Attaker steal liquidity tokens by removing liquidity multiple times
    ## DESCRIPTION
    The PlunderPair protocol employs a Uniswap system in which users can provide liquidity to mint Lp tokens  and earn fees from users swap, However when `removeLiquidityETHSupportingFeeOnTransferToken` or a other removeLiquidity function is called it tries to burn liquidity from the protocol address instead of the `to` address which is the liqidity provider, This would allow an attacker drain Plunderswap pool by providing liquidity and repeating it to drain protocol

      https://github.com/Plunderswap/plunderswap-core/blob/18e823c151c57521b8659ebc72b41529a5fbfe53/contracts/PlunderPair.sol#L184
    ```solidity
    function burn(address to) external lock returns (uint amount0, uint amount1) {
    (uint112 _reserve0, uint112 _reserve1, ) = getReserves(); // gas savings
    address _token0 = token0; // gas savings
    address _token1 = token1; // gas savings
    uint balance0 = IERC20(_token0).balanceOf(address(this));
    uint balance1 = IERC20(_token1).balanceOf(address(this));
    uint liquidity = balanceOf[address(this)];

    bool feeOn = _mintFee(_reserve0, _reserve1);
    uint _totalSupply = totalSupply; // gas savings, must be defined here since totalSupply can update in _mintFee
    amount0 = liquidity.mul(balance0) / _totalSupply; // using balances ensures pro-rata distribution
    amount1 = liquidity.mul(balance1) / _totalSupply; // using balances ensures pro-rata distribution
    require(
      amount0 > 0 && amount1 > 0,
      "Plunderswap: INSUFFICIENT_LIQUIDITY_BURNED"
    );
     @>  _burn(address(this), liquidity);
    ```
    As we can see in my pointerprotocol wrongly burns tokens from address this intstead of to address, this can allow an attacker who added liquidity repeatedly burn liquidity tokens of protocol while getting free tokens
    ## IMPACT
 Attacker can drain liquidity tokens of Plunder swap
    ## Recommended Mitigation
    Burn tokens from `to` address

    
  ## CRICITICAL- 2 ATTACKER can deposit `fake tokens` to mint plunder Lp tokens
  ## Description
  Plunder swap allows user to provide liquidity of any tokens to get mint Lp tokens, However it fails to vallidate the tokens being provided this would allow an attacker deploy fake erc20 tokens that would have a pair on Uniswap but have zero value, Then mint free Lp tokens and use them to withdraw ETH tokens from the pool since protocol allows user to provide liquidity on any token and remove liquidity of other tokens that the Liquidity proider wants. This would allow an attacker remove liquidity at zero cost with fake tokens

https://github.com/Plunderswap/plunderswap-core/blob/18e823c151c57521b8659ebc72b41529a5fbfe53/contracts/PlunderRouter.sol#L43C2-L43C3

  ```solidity
  function _addLiquidity(
    address tokenA,
    address tokenB,
    uint256 amountADesired,
    uint256 amountBDesired,
    uint256 amountAMin,
    uint256 amountBMin
  ) internal virtual returns (uint256 amountA, uint256 amountB) {
    // create the pair if it doesn't exist yet
    if (IPlunderFactory(factory).getPair(tokenA, tokenB) == address(0)) {
 @>     IPlunderFactory(factory).createPair(tokenA, tokenB);
    }
    (uint256 reserveA, uint256 reserveB) = PlunderLibrary.getReserves(
      factory,
      tokenA,
      tokenB
    );
    if (reserveA == 0 && reserveB == 0) {
      (amountA, amountB) = (amountADesired, amountBDesired);
```
As we can see in the pointer if the pair of the fake tokens is not found protocol would deploy new pair for them , this opens a vulnerable path for attacker to deploy new fresh fake tokens add them to the liquidity pool, And Plunder swap would mint LP tokens to the attackers address for free. After he selects actula tokens with high value burns the Lp tokens and walk away with free tokens!.

## Impact
Attackers would gain free liquidity tokens from fake ERC20 tokens

## Recommended Mitigation
Employ a selected whitelist tokens
    
