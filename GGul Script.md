# GGul Script

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GGul : MonoBehaviour
{
    public float MaxSpeed;
    public float JumpPower;
    Rigidbody2D rigid;
    SpriteRenderer spriteRenderer;
    Animator anim;

    private void Awake()
    {
        rigid = GetComponent<Rigidbody2D>();
        spriteRenderer = GetComponent<SpriteRenderer>();
        anim = GetComponent<Animator>();
    }

    //단발성 키 입력은 FixedUpdate말고 Update에
    private void Update()
    {
        if(Input.GetButtonDown("Jump") && !anim.GetBool("isJump"))
        {
            rigid.AddForce(Vector2.up * JumpPower, ForceMode2D.Impulse);
            anim.SetBool("isJump", true);
        }

        //PlayerStop
        //키보드 뗐을 때 속력 급격히 감속.
        if (Input.GetButtonUp("Horizontal"))
            rigid.velocity = new Vector2(rigid.velocity.normalized.x * 0.5f, rigid.velocity.y);

        //방향전환
        if (Input.GetButtonDown("Horizontal"))
            spriteRenderer.flipX = Input.GetAxisRaw("Horizontal") == -1;
    }


    private void FixedUpdate()
    {
        float h = Input.GetAxisRaw("Horizontal");
        //MaxSpeedLimit
        rigid.AddForce(Vector2.right * h, ForceMode2D.Impulse);

        //velocity : rigidbody의 현재 속도
        if (rigid.velocity.x > MaxSpeed)
        {
            //속도 최댓값 이하로 고정.
            //y 속도 0으로 설정하면 그냥 멈춰버림.

            //RightMaxSpeed
            rigid.velocity = new Vector2(MaxSpeed, rigid.velocity.y);

        }
        else if (rigid.velocity.x < -MaxSpeed)
        {
            //LeftMaxSpeed
            rigid.velocity = new Vector2(-MaxSpeed, rigid.velocity.y);
        }


        //내려가고 있을 때만 ray 투과.
        if (rigid.velocity.y < 0)
        {
            //RayCast
            //Landing Platform
            //DrawRay : 에디터 상에서만 Ray를 그려주는 함수
            Debug.DrawRay(rigid.position, Vector3.down * 2, new Color(0, 1, 0));

            //RayCastHit : Ray에 닿은 오브젝트 정보 저장
            RaycastHit2D RayHit = Physics2D.Raycast(rigid.position, Vector3.down * 2, 1, LayerMask.GetMask("Platform"));


            //충돌 판정 ** 주의 **
            //collider null 값 아닌 경우 내용물이 있는 것.
            if (RayHit.collider != null)
            {
                //플레이어가 바닥에 완전히 착지했는지 여부
                //distance : Ray에 닿았을 때의 거리
                if (RayHit.distance < 1)
                {
                    //충돌 확인 : Debug.Log(RayHit.collider.name);
                    //착지했을 때 불값 변경
                    anim.SetBool("isLanding", true);
                    anim.SetBool("isJump", false);
                    Invoke("Landing", 0.4f);
                }
            }
        }

    }
    void Landing()
    {
        anim.SetBool("isLanding", false);
    }
}
```

