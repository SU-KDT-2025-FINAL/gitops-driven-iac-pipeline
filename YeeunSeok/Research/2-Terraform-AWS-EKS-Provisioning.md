# Terraform을 이용한 AWS EKS 클러스터 프로비저닝 고도화 가이드

---

## 1. 개념 및 아키텍처

### 1.1 EKS란?
- AWS에서 제공하는 완전관리형 Kubernetes 서비스
- 고가용성, 자동 업그레이드, 통합 IAM, VPC 네트워킹 등 지원

### 1.2 Terraform이란?
- HashiCorp에서 개발한 Infrastructure as Code(IaC) 도구
- 선언적 언어(HCL)로 인프라 리소스 정의 및 관리
- AWS, GCP, Azure 등 다양한 클라우드 지원

### 1.3 EKS 프로비저닝 아키텍처
- VPC, Subnet, IAM, EKS Cluster, Node Group 등 여러 리소스가 상호작용
- 모듈화된 구조로 관리 권장

---

## 2. 실습: EKS 클러스터 프로비저닝

### 2.1 프로젝트 구조 예시
```
terraform-eks/
  main.tf
  variables.tf
  outputs.tf
  provider.tf
  modules/
    vpc/
    eks/
    nodegroup/
```

### 2.2 Provider 및 Backend 설정
```hcl
provider "aws" {
  region = var.aws_region
}

terraform {
  backend "s3" {
    bucket = "my-terraform-state-bucket"
    key    = "eks/terraform.tfstate"
    region = "ap-northeast-2"
    dynamodb_table = "my-terraform-lock"
  }
}
```

### 2.3 VPC 및 네트워크 구성
- 공식 VPC 모듈 활용 권장
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  name = "eks-vpc"
  cidr = "10.0.0.0/16"
  azs             = ["ap-northeast-2a", "ap-northeast-2c"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.101.0/24", "10.0.102.0/24"]
  enable_nat_gateway = true
}
```

### 2.4 EKS 클러스터 및 노드 그룹 생성
```hcl
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  version         = "~> 20.0"
  cluster_name    = "my-eks-cluster"
  cluster_version = "1.29"
  subnet_ids      = module.vpc.private_subnets
  vpc_id          = module.vpc.vpc_id

  eks_managed_node_groups = {
    default = {
      desired_size = 2
      max_size     = 3
      min_size     = 1
      instance_types = ["t3.medium"]
    }
  }
}
```

### 2.5 실행 및 검증
```bash
terraform init
terraform plan
terraform apply
```
- `aws eks update-kubeconfig --region <region> --name <cluster_name>`로 kubeconfig 설정
- `kubectl get nodes`로 정상 생성 확인

---

## 3. 모듈화 및 확장
- VPC, EKS, NodeGroup, IAM 등 각 리소스를 별도 모듈로 분리
- 변수와 output을 적극 활용해 재사용성 및 가독성 향상
- 예시: `modules/eks/variables.tf`, `modules/eks/outputs.tf`

---

## 4. 실전 예제: GitOps와 연계
- EKS 클러스터 프로비저닝 후 ArgoCD 등 GitOps 도구와 연동
- Terraform output을 CI 파이프라인에서 활용해 자동화
- 예시: output으로 kubeconfig, cluster endpoint, 인증 정보 등 노출

---

## 5. 베스트 프랙티스
- 상태 파일은 S3 + DynamoDB로 관리(잠금 및 이력)
- 최소 권한 원칙의 IAM 정책 설계
- 태그 일관성 부여(비용, 관리, 보안)
- 모듈 버전 고정 및 주기적 업데이트
- `terraform validate`, `terraform fmt`, `tflint` 등 도구로 품질 관리

---

## 6. 트러블슈팅
- IAM 권한 부족: AssumeRole, 정책 누락 확인
- VPC/Subnet 미설정: 서브넷, 라우팅, NAT Gateway 등 네트워크 설정 점검
- 버전 불일치: Terraform, AWS Provider, EKS 모듈 버전 호환성 확인
- 노드 그룹 생성 실패: EC2 Quota, AMI, 인스턴스 타입 가용성 확인

---

## 7. 참고 자료
- [terraform-aws-modules/eks/aws 공식 문서](https://github.com/terraform-aws-modules/terraform-aws-eks)
- [AWS EKS 공식 문서](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Terraform 공식 문서](https://developer.hashicorp.com/terraform/docs)
- [EKS Best Practices Guide](https://aws.github.io/aws-eks-best-practices/) 