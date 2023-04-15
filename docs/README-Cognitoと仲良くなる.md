# Cognitoと仲良くなる

## これはなに?

AWSのCognitoと仲良くなって、GolangとReactでゴリゴリ開発する。
そういうお気持ちの、記録。

## 構成

![image](https://user-images.githubusercontent.com/14959592/232183094-8dc62130-36a4-41dd-adfe-3b4b8814cf72.png)

## Cognito APIユースケース

| AWSサービス| 操作（日本語名）| AWS API名 | 権限レベル | 
| --- | --- |  --- |  --- |
| Amazon Cognito​（Cognito User Pool）| サインイン| InitiateAuth| 一般|
| | サインイン-チャレンジレスポンス| RespondToAuthChallenge| 一般 |
| | サインアウト（全デバイス）| GlobalSignOut| 一般 |
| | サインアウト（全デバイス）（管理者実行）| AdminUserGlobalSignOut| 管理者 |
| | サインアップ| SignUp| 一般 |
| | サインアップ確認（管理者実行）| AdminConfirmSignUp| 管理者 |
| | サインイン-チャレンジレスポンス（管理者実行）| AdminRespondToAuthChallenge| 管理者|
| | サインイン（管理者実行）| AdminInitiateAuth| 管理者| 
| | サインイン無効化（管理者実行）| AdminDisableProviderForUser| 管理者| 
| | ユーザーパスワードリセット（申請）| ForgotPassword| 一般| 
| | ユーザーパスワードリセット（確認）| ConfirmForgotPassword| 一般| 
| | パスワード変更| ChangePassword| 一般| 
| | ユーザ-IdPリンク（管理者実行）| AdminLinkProviderForUser| 管理者| 
| | ユーザMFA(TOTP)セットアップ開始要求| AssociateSoftwareToken| 一般| 
| | ユーザMFA設定変更| SetUserMFAPreference| 一般| 
| | ユーザMFA設定変更(SMS MFA専用) ※非推奨| SetUserSettings| 一般| 
| | ユーザMFA設定変更(SMS MFA専用)（管理者実行）| AdminSetUserSettings| 管理者| 
| | ユーザMFA設定変更（管理者実行）| AdminSetUserMFAPreference| 管理者| 
| | ユーザアクティビティ履歴取得（管理者実行）| AdminListUserAuthEvents| 管理者| 
| | ユーザーデバイスリセット（確認）| ConfirmDevice| 一般| 
| | ユーザーデバイスリセット（申請）| ForgetDevice| 一般| 
| | ユーザーデバイス取得| GetDevice| 一般| 
| | ユーザーデバイス登録解除（申請）（管理者実行）| AdminForgetDevice| 管理者| 
| | ユーザー新規登録（確認）| ConfirmSignUp| 一般| 
| | ユーザー登録時確認コード再送| ResendConfirmationCode| 一般| 
| | ユーザグループ関連付け解除（管理者実行）| AdminRemoveUserFromGroup| 管理者| 
| | ユーザグループ関連付け登録（管理者実行）| AdminAddUserToGroup| 管理者| 
| | ユーザサインインデバイスリスト取得| ListDevices| 一般| 
| | ユーザデバイスステータス更新| UpdateDeviceStatus| 一般| 
| | ユーザデバイスリスト取得（管理者実行）| AdminListDevices| 管理者| 
| | ユーザデバイス更新（管理者実行）| AdminUpdateDeviceStatus| 管理者| 
| | ユーザデバイス取得（管理者実行）| AdminGetDevice| 管理者| 
| | ユーザトークン検証(TOTP MFA用)| VerifySoftwareToken| 一般| 
| | ユーザパスワードリセット（管理者実行）| AdminResetUserPassword| 管理者| 
| | ユーザパスワード設定（管理者実行）| AdminSetUserPassword| 管理者| 
| | ユーザ作成（管理者実行）| AdminCreateUser| 管理者| 
| | ユーザ削除（管理者実行）| AdminDeleteUser| 管理者| 
| | ユーザ削除（管理者実行）| AdminDeleteUserAttributes| 管理者| 
| | ユーザ所属グループリスト取得（管理者実行）| AdminListGroupsForUser| 管理者| 
| | ユーザ情報削除（退会）| DeleteUser| 一般| 
| | ユーザ情報取得| GetUser| 一般| 
| | ユーザ情報取得（管理者実行）| AdminGetUser| 管理者| 
| | ユーザ属性検証| VerifyUserAttribute| 一般| 
| | ユーザ属性検証コード生成（コード送付）| GetUserAttributeVerificationCode| 一般| 
| | ユーザ属性更新（管理者実行）| AdminUpdateUserAttributes| 管理者| 
| | ユーザ属性削除| DeleteUserAttributes| 一般| 
| | ユーザ属性情報更新| UpdateUserAttributes| 一般| 
| | ユーザ無効化（管理者実行）| AdminDisableUser| 管理者| 
| | ユーザ有効化（管理者実行）| AdminEnableUser| 管理者| 
| | 個別リフレッシュトークン無効化| RevokeToken| 一般| 
| | 認証イベントフィードバック（Amazon Cognito advanced security）| UpdateAuthEventFeedback| 一般| 
| | 認証イベントフィードバック（管理者実行）| AdminUpdateAuthEventFeedback| 管理者| 
| Amazon Cognito​（Cognito Identity Pools）| | DeleteIdentities| 管理者| 
| | 一時的な認証情報取得| GetCredentialsForIdentity| 一般| 
| | Cognito Identity PoolsのユーザIdentity取得| GetId| 一般| 
| | | GetIdentityPoolRoles| 一般| 
| | Cognito Identity Pools利用時OpenIDトークン取得| GetOpenIdToken| 一般| 
| | | GetOpenIdTokenForDeveloperIdentity| 管理者| 
| | | UnlinkIdentity| 一般| 
| | Cognito JWTトークン検証| （APIではない）| ー | 

## IDトークン構造

### ヘッダ

| 情報 | 説明 |
| --- | --- |
| kid | 公開鍵のID |
| alg | 暗号化アルゴリズム(ここではRS256) |

###  ペイロード

| 情報 | 説明 |
| --- | --- |
| iss | トークンの発行者(ここではCognitoのユーザープール) |
| aud | トークンの受信者(ここではCognitoのクライアントアプリケーション) |
| token_use | トークンの用途("id"はIDトークン、"access"はアクセストークン) |
| sub | 認証されたユーザーの固有識別子(UUID) |
| cognito:username | Cognitoでランダムに生成されたユーザー名 |
| email | ユーザーのメールアドレス |
| email_verified | メールアドレスが確認されたかどうかのフラグ |
| given_name | ユーザーの名 |
| jti | トークンのユニークキー |
| origin_jti | IDトークンの場合、元となるアクセストークンのjti |
| auth_time | 認証された時刻(1970年1月1日からの秒数) |
| exp | トークンの有効期限(1970年1月1日からの秒数) |
| iat | トークンの発行時刻(1970年1月1日からの秒数) |


## Golang


```golang

package jwt

import (
	"context"
	"errors"
	"fmt"
	"os"
	"time"
	"encoding/json"
	jwt "github.com/golang-jwt/jwt"
	jwk "github.com/lestrrat-go/jwx/jwk"
	backoff "github.com/lestrrat-go/backoff/v2"
)

// TODO: 検証時の暫定処理です(本開発時は別途設計を行う必要があります)↓↓
// Cognitoから返されるTokenの構造に合わせてStructを定義する
type CognitoJwtClaims struct {
	Sub string `json:"sub"`
	Username  string `json:"cognito:username"`
	Email string `json:"email"`
	EmailVerified bool `json:"email_verified"`
	Expire  int64 `json:"exp"`
}

func Verify(tokenString string) (CognitoJwtClaims, error) {
	var jwtClaims CognitoJwtClaims
	claims := jwt.MapClaims{}
	// see: AWS Amazon Cognito JSON ウェブトークンの署名を復号して検証するには https://aws.amazon.com/jp/premiumsupport/knowledge-center/decode-verify-cognito-json-token/
	// see: awslabs/aws-jwt-verify cognito-verifier(typescript) https://github.com/awslabs/aws-jwt-verify/blob/main/src/cognito-verifier.ts
	publicKeysURL := os.Getenv("COGNITO_USER_POOL_PUBLIC_KEY")
	fetchOption  := jwk.WithFetchBackoff(backoff.Null())
	publicKeySet, err := jwk.Fetch(context.TODO(), publicKeysURL, fetchOption)
	if err != nil {
		fmt.Println(tokenString, err)
		return jwtClaims, err
	}
	// Cognitoパブリックキーを使って、Cognitoが発行したJSON Web Token(ここではIDトークン)の復号・検証
	token, err := jwt.ParseWithClaims(tokenString, &claims, func(token *jwt.Token) (interface{}, error) {

			// ユーザープールは、SHA-256 による RSA 署名である RS256 暗号化アルゴリズムを使用 - https://docs.aws.amazon.com/ja_jp/cognito/latest/developerguide/amazon-cognito-user-pools-using-the-id-token.html
			_, ok := token.Method.(*jwt.SigningMethodRSA);
			if !ok {
				return nil, fmt.Errorf("Unexpected signing method: %v", token.Header["alg"])
			}

			kid, ok := token.Header["kid"].(string)
			if !ok {
					return nil, errors.New("kid header not found")
			}

			// "kid" must be present in the public keys set （アプリケーションで独自にキャッシュした場合は、キャッシュ内に対応するパブリックキーが見つからければキャッシュクリア＋再取得した上で判断する）
			key, ok := publicKeySet.LookupKeyID(kid);
			if  !ok {
					return nil, fmt.Errorf("key %v not found", kid)
			}

			var tokenKey interface{}
			if err := key.Raw(&tokenKey); err != nil {
				return nil, errors.New("failed to create token key")
			}
			return tokenKey, nil
	})

	if err != nil {
		fmt.Println(tokenString, err)
		return jwtClaims, err
	}
	// 一度JSONへ変換
	jsonString, err := json.Marshal(token.Claims)
	if err != nil {
		fmt.Println("JSON Marshal error: ", err)
		return jwtClaims, err
	}

	// 定義したStructへ変換
	if err := json.Unmarshal(jsonString, &jwtClaims); err != nil {
		fmt.Println("jsonunmarshal error", err)
		return jwtClaims, err
	}
	
	// check expire time https://openid.net/specs/openid-connect-core-1_0.html#Claims
	now := time.Now()
	if time.Unix(jwtClaims.Expire, 0).Before(now) || time.Unix(jwtClaims.Expire, 0).Equal(now) {
		err = errors.New("token is expired")
		return jwtClaims, err
	
	return jwtClaims, nil
}
```
