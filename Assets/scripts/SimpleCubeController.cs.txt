using UnityEngine;
using UnityEngine.InputSystem;

[RequireComponent(typeof(Rigidbody))]
public class SimpleCubeController : MonoBehaviour
{
    [Header("Movement")]
    [SerializeField] private float moveSpeed = 5f;
    [SerializeField] private float sprintMultiplier = 1.8f;

    [Header("Jump")]
    [SerializeField] private float jumpHeight = 1.5f;
    [SerializeField] private float groundCheckDistance = 0.6f;
    [SerializeField] private LayerMask groundLayers = ~0;

    private Rigidbody rb;
    private Vector3 moveInput;
    private bool jumpQueued;

    private void Awake()
    {
        rb = GetComponent<Rigidbody>();
        rb.freezeRotation = true;
    }

    private void Update()
    {
        if (Keyboard.current == null)
        {
            moveInput = Vector3.zero;
            return;
        }

        Vector2 input = Vector2.zero;

        if (Keyboard.current.wKey.isPressed)
        {
            input.y += 1f;
        }
        if (Keyboard.current.sKey.isPressed)
        {
            input.y -= 1f;
        }
        if (Keyboard.current.aKey.isPressed)
        {
            input.x -= 1f;
        }
        if (Keyboard.current.dKey.isPressed)
        {
            input.x += 1f;
        }

        moveInput = new Vector3(input.x, 0f, input.y).normalized;

        if (Keyboard.current.spaceKey.wasPressedThisFrame && IsGrounded())
        {
            jumpQueued = true;
        }
    }

    private void FixedUpdate()
    {
        float currentSpeed = moveSpeed;

        if (Keyboard.current != null)
        {
            if (Keyboard.current.leftShiftKey.isPressed || Keyboard.current.rightShiftKey.isPressed)
            {
                currentSpeed *= sprintMultiplier;
            }
        }

        Vector3 velocity = rb.linearVelocity;
        velocity.x = moveInput.x * currentSpeed;
        velocity.z = moveInput.z * currentSpeed;
        rb.linearVelocity = velocity;

        if (jumpQueued)
        {
            jumpQueued = false;

            Vector3 jumpVelocity = rb.linearVelocity;
            jumpVelocity.y = Mathf.Sqrt(jumpHeight * -2f * Physics.gravity.y);
            rb.linearVelocity = jumpVelocity;
        }
    }

    private bool IsGrounded()
    {
        return Physics.Raycast(
            transform.position,
            Vector3.down,
            groundCheckDistance,
            groundLayers,
            QueryTriggerInteraction.Ignore
        );
    }
}
